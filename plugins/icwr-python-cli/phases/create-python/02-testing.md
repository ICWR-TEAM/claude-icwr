# Testing Python CLI Tools for Exploit/Scanning

This phase covers comprehensive testing strategies for security CLI tools to ensure reliability, safety, and effectiveness.

## Testing Framework Setup

### 1. Unit Testing with pytest
```python
# test_security_tool.py
import pytest
from unittest.mock import Mock, patch, MagicMock
import sys
import os

# Add the tool's directory to Python path
sys.path.insert(0, os.path.dirname(os.path.dirname(__file__)))

from your_tool import SecurityTool

class TestSecurityTool:
    def setup_method(self):
        """Setup before each test"""
        self.tool = SecurityTool.__new__(SecurityTool)  # Create without __init__

    def test_validate_target_valid_ip(self):
        """Test IP address validation"""
        assert self.tool.validate_target("192.168.1.1") == True
        assert self.tool.validate_target("127.0.0.1") == True

    def test_validate_target_invalid_ip(self):
        """Test invalid IP address"""
        assert self.tool.validate_target("999.999.999.999") == False
        assert self.tool.validate_target("invalid.ip") == False

    @patch('socket.gethostbyname')
    def test_validate_target_hostname(self, mock_gethostbyname):
        """Test hostname validation"""
        mock_gethostbyname.return_value = "192.168.1.1"
        assert self.tool.validate_target("example.com") == True

        mock_gethostbyname.side_effect = socket.gaierror
        assert self.tool.validate_target("invalid.domain") == False
```

### 2. Integration Testing
```python
# test_integration.py
import subprocess
import sys
import tempfile
import os

class TestCLIIntegration:
    def test_cli_help_output(self):
        """Test CLI help output"""
        result = subprocess.run(
            [sys.executable, 'your_tool.py', '--help'],
            capture_output=True,
            text=True
        )
        assert result.returncode == 0
        assert 'usage:' in result.stdout.lower()
        assert 'target' in result.stdout.lower()

    def test_cli_invalid_target(self):
        """Test CLI with invalid target"""
        result = subprocess.run(
            [sys.executable, 'your_tool.py', '-t', 'invalid.target'],
            capture_output=True,
            text=True
        )
        assert result.returncode != 0
        assert 'invalid' in result.stderr.lower()

    def test_cli_missing_required_args(self):
        """Test CLI with missing required arguments"""
        result = subprocess.run(
            [sys.executable, 'your_tool.py'],
            capture_output=True,
            text=True
        )
        assert result.returncode != 0
        assert 'required' in result.stderr.lower()
```

## Network Testing

### 1. Mock Network Operations
```python
# test_network.py
import pytest
from unittest.mock import Mock, patch, MagicMock
import socket

class TestNetworkOperations:
    @patch('socket.socket')
    def test_port_scan_success(self, mock_socket):
        """Test successful port scan"""
        mock_sock = MagicMock()
        mock_sock.connect_ex.return_value = 0  # Port open
        mock_socket.return_value = mock_sock

        tool = SecurityTool.__new__(SecurityTool)
        tool.target_ip = "127.0.0.1"
        tool.args = Mock()
        tool.args.timeout = 1.0

        # Test port scanning logic
        result = tool.scan_port(80)
        assert 80 in tool.open_ports

    @patch('socket.socket')
    def test_port_scan_closed(self, mock_socket):
        """Test closed port scan"""
        mock_sock = MagicMock()
        mock_sock.connect_ex.return_value = 1  # Port closed
        mock_socket.return_value = mock_sock

        tool = SecurityTool.__new__(SecurityTool)
        tool.target_ip = "127.0.0.1"
        tool.args = Mock()
        tool.args.timeout = 1.0

        result = tool.scan_port(80)
        assert 80 not in tool.open_ports
```

### 2. Test Server Setup
```python
# test_server.py
import socket
import threading
import time
import pytest

class TestServer:
    def __init__(self, host='127.0.0.1', port=8888):
        self.host = host
        self.port = port
        self.server_socket = None
        self.thread = None

    def start(self):
        """Start test server"""
        self.server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server_socket.bind((self.host, self.port))
        self.server_socket.listen(5)

        self.thread = threading.Thread(target=self._serve)
        self.thread.daemon = True
        self.thread.start()
        time.sleep(0.1)  # Allow server to start

    def _serve(self):
        """Server loop"""
        while True:
            try:
                client, addr = self.server_socket.accept()
                client.close()
            except:
                break

    def stop(self):
        """Stop test server"""
        if self.server_socket:
            self.server_socket.close()

@pytest.fixture
def test_server():
    server = TestServer()
    server.start()
    yield server
    server.stop()

class TestWithLiveServer:
    def test_scan_open_port(self, test_server):
        """Test scanning an actually open port"""
        tool = SecurityTool.__new__(SecurityTool)
        tool.target_ip = "127.0.0.1"
        tool.args = Mock()
        tool.args.timeout = 1.0

        tool.scan_port(test_server.port)
        assert test_server.port in tool.open_ports
```

## Security Testing

### 1. Input Validation Testing
```python
# test_security.py
import pytest

class TestInputValidation:
    def test_sql_injection_prevention(self):
        """Test prevention of SQL injection"""
        tool = SecurityTool.__new__(SecurityTool)

        # Test various injection attempts
        malicious_inputs = [
            "'; DROP TABLE users; --",
            "1' OR '1'='1",
            "<script>alert('xss')</script>",
            "../../../etc/passwd",
            "127.0.0.1; rm -rf /",
        ]

        for malicious_input in malicious_inputs:
            # Should not execute dangerous commands
            assert not tool.is_input_safe(malicious_input)

    def test_command_injection_prevention(self):
        """Test prevention of command injection"""
        tool = SecurityTool.__new__(SecurityTool)

        dangerous_commands = [
            "127.0.0.1; ls -la",
            "127.0.0.1 | cat /etc/passwd",
            "127.0.0.1 && echo 'hacked'",
        ]

        for cmd in dangerous_commands:
            with patch('subprocess.run') as mock_run:
                tool.safe_execute_command(cmd)
                # Should not execute the command
                mock_run.assert_not_called()
```

### 2. Rate Limiting Testing
```python
# test_rate_limiting.py
import time
import pytest
from unittest.mock import patch

class TestRateLimiting:
    def test_rate_limit_enforcement(self):
        """Test that rate limiting works"""
        tool = SecurityTool.__new__(SecurityTool)
        tool.rate_limiter = Mock()

        # Simulate rapid requests
        for i in range(100):
            tool.make_request("127.0.0.1")

        # Rate limiter should have been triggered
        assert tool.rate_limiter.is_allowed.call_count < 100
```

## Performance Testing

### 1. Load Testing
```python
# test_performance.py
import time
import pytest
from concurrent.futures import ThreadPoolExecutor

class TestPerformance:
    def test_concurrent_scanning(self):
        """Test performance under concurrent load"""
        tool = SecurityTool.__new__(SecurityTool)
        tool.target_ip = "127.0.0.1"
        tool.args = Mock()
        tool.args.threads = 50
        tool.args.timeout = 0.1

        ports = list(range(1, 1001))  # Test 1000 ports

        start_time = time.time()

        with ThreadPoolExecutor(max_workers=tool.args.threads) as executor:
            executor.map(tool.scan_port, ports)

        end_time = time.time()
        duration = end_time - start_time

        # Should complete within reasonable time
        assert duration < 30  # 30 seconds max
        print(f"Scanned 1000 ports in {duration:.2f} seconds")

    def test_memory_usage(self):
        """Test memory usage during large scans"""
        import psutil
        import os

        process = psutil.Process(os.getpid())
        initial_memory = process.memory_info().rss / 1024 / 1024  # MB

        tool = SecurityTool.__new__(SecurityTool)
        # Perform large scan operation

        final_memory = process.memory_info().rss / 1024 / 1024  # MB
        memory_increase = final_memory - initial_memory

        # Memory increase should be reasonable
        assert memory_increase < 100  # Less than 100MB increase
```

## Error Handling Testing

### 1. Exception Testing
```python
# test_error_handling.py
import pytest
from unittest.mock import patch, Mock

class TestErrorHandling:
    def test_network_timeout(self):
        """Test handling of network timeouts"""
        tool = SecurityTool.__new__(SecurityTool)

        with patch('socket.socket') as mock_socket:
            mock_sock = Mock()
            mock_sock.connect_ex.side_effect = socket.timeout
            mock_socket.return_value = mock_sock

            # Should handle timeout gracefully
            result = tool.scan_port(80)
            assert result is None  # Or appropriate error handling

    def test_connection_refused(self):
        """Test handling of connection refused"""
        tool = SecurityTool.__new__(SecurityTool)

        with patch('socket.socket') as mock_socket:
            mock_sock = Mock()
            mock_sock.connect_ex.side_effect = ConnectionRefusedError
            mock_socket.return_value = mock_sock

            result = tool.scan_port(80)
            assert result is None

    def test_keyboard_interrupt(self):
        """Test handling of user interruption"""
        tool = SecurityTool.__new__(SecurityTool)

        with patch.object(tool, 'run_scan', side_effect=KeyboardInterrupt):
            with pytest.raises(SystemExit):
                tool.run()
```

## Test Configuration

### pytest.ini
```ini
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts =
    --verbose
    --tb=short
    --strict-markers
    --disable-warnings
markers =
    slow: marks tests as slow (deselect with '-m "not slow"')
    integration: marks tests as integration tests
    network: marks tests that require network access
```

### Test Requirements
```txt
# requirements-test.txt
pytest>=7.0.0
pytest-mock>=3.10.0
pytest-cov>=4.0.0
psutil>=5.9.0
requests>=2.28.0
```

## Running Tests

### Basic Test Execution
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_security_tool.py

# Run tests with coverage
pytest --cov=your_tool --cov-report=html

# Run slow tests only
pytest -m slow

# Run tests excluding network tests
pytest -m "not network"
```

### CI/CD Integration
```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install -r requirements-test.txt
    - name: Run tests
      run: pytest --cov=your_tool --cov-report=xml
    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

## Best Practices

1. **Test Isolation**: Each test should be independent
2. **Mock External Dependencies**: Use mocks for network calls, file I/O
3. **Test Edge Cases**: Cover boundary conditions and error scenarios
4. **Performance Benchmarks**: Set acceptable performance thresholds
5. **Security Testing**: Include penetration testing in CI/CD
6. **Continuous Testing**: Run tests on every code change
7. **Test Documentation**: Document test cases and expected behavior

## Safety Considerations

- **Test Environments**: Only run tests in isolated environments
- **Mock Dangerous Operations**: Never execute real exploits in tests
- **Rate Limiting**: Implement rate limiting in test scenarios
- **Cleanup**: Ensure proper cleanup after tests
- **Ethical Testing**: Follow responsible disclosure and legal guidelines