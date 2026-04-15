---
name: create-python
description: Create secure, efficient Python command-line tools for security testing, exploit development, and network scanning operations. Use when: building Python CLI tools for port scanning, exploits, reconnaissance, or custom security analysis.
---

# Create Python CLI Tool for Security Operations

This skill guides you through creating robust Python command-line tools for security testing, exploit development, and network scanning.

## Phase 1: Tool Design and Implementation

### Structure Requirements

1. **OOP Design with Single Class**
   - Use object-oriented programming with one main class
   - Class handles initialization, argument parsing, and execution
   - Pattern: `__init__` → `setup_arguments()` → `run()`

2. **CLI Framework with argparse**
   ```python
   import argparse
   import sys

   class SecurityTool:
       def __init__(self):
           self.parser = argparse.ArgumentParser(
               description="Security scanning and exploit tool",
               formatter_class=argparse.RawDescriptionHelpFormatter
           )
           self.setup_arguments()
           self.args = self.parser.parse_args()
           self.run()

       def setup_arguments(self):
           self.parser.add_argument(
               '-t', '--target',
               required=True,
               help='Target IP/hostname/domain'
           )
           self.parser.add_argument(
               '-p', '--port',
               type=int,
               default=80,
               help='Target port (default: 80)'
           )
           self.parser.add_argument(
               '--verbose',
               action='store_true',
               help='Enable verbose output'
           )
   ```

### Core Components Implementation

#### Target Validation
```python
def validate_target(self, target):
    """Validate target format and reachability"""
    import ipaddress
    import socket

    try:
        ipaddress.ip_address(target)
        return True
    except ValueError:
        try:
            socket.gethostbyname(target)
            return True
        except socket.gaierror:
            return False
```

#### Scanning Module
```python
def scan_target(self):
    """Perform security scanning"""
    if not self.validate_target(self.args.target):
        print(f"[-] Invalid target: {self.args.target}")
        return

    print(f"[+] Scanning target: {self.args.target}:{self.args.port}")
    # Implement scanning logic
```

#### Exploit Module
```python
def exploit_target(self):
    """Execute exploit against target"""
    print(f"[+] Attempting exploit on {self.args.target}")
    # Implement exploit logic
```

### Error Handling and Logging
```python
import logging
import traceback

class SecurityTool:
    def __init__(self):
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s'
        )
        self.logger = logging.getLogger(__name__)

    def safe_execute(self, func, *args, **kwargs):
        """Safely execute functions with error handling"""
        try:
            return func(*args, **kwargs)
        except KeyboardInterrupt:
            print("\n[!] Operation cancelled by user")
            sys.exit(0)
        except Exception as e:
            self.logger.error(f"Error in {func.__name__}: {str(e)}")
            if self.args.verbose:
                traceback.print_exc()
            return None
```

### Output Formatting
```python
def print_banner(self):
    """Display tool banner"""
    banner = """
    ===========================================
           Security Tool v1.0
           Author: HarshXor
    ===========================================
    """
    print(banner)

def print_result(self, result, success=True):
    """Format and display results"""
    status = "[+]" if success else "[-]"
    print(f"{status} {result}")
```

### Main Execution Flow
```python
def run(self):
    """Main execution method"""
    self.print_banner()

    if not self.args.target:
        self.parser.print_help()
        sys.exit(1)

    if not self.validate_target(self.args.target):
        self.print_result(f"Invalid target: {self.args.target}", False)
        sys.exit(1)

    try:
        if hasattr(self.args, 'scan') and self.args.scan:
            self.safe_execute(self.scan_target)
        elif hasattr(self.args, 'exploit') and self.args.exploit:
            self.safe_execute(self.exploit_target)
        else:
            self.parser.print_help()
    except Exception as e:
        self.print_result(f"Execution failed: {str(e)}", False)
        sys.exit(1)

if __name__ == "__main__":
    SecurityTool()
```

## Example: Port Scanner Implementation

```python
#!/usr/bin/env python3
import argparse
import socket
import sys
import threading
import time
from concurrent.futures import ThreadPoolExecutor

class PortScanner:
    def __init__(self):
        self.parser = argparse.ArgumentParser(description="Simple Port Scanner")
        self.setup_arguments()
        self.args = self.parser.parse_args()
        self.open_ports = []
        self.run()

    def setup_arguments(self):
        self.parser.add_argument('-t', '--target', required=True, help='Target IP or hostname')
        self.parser.add_argument('-p', '--ports', default='1-1024', help='Port range (default: 1-1024)')
        self.parser.add_argument('--threads', type=int, default=100, help='Number of threads')
        self.parser.add_argument('--timeout', type=float, default=1.0, help='Connection timeout')

    def validate_target(self, target):
        try:
            socket.gethostbyname(target)
            return True
        except socket.gaierror:
            return False

    def scan_port(self, port):
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(self.args.timeout)
            result = sock.connect_ex((self.target_ip, port))
            sock.close()
            if result == 0:
                self.open_ports.append(port)
                print(f"[+] Port {port} is open")
        except:
            pass

    def parse_ports(self, port_range):
        ports = []
        for part in port_range.split(','):
            if '-' in part:
                start, end = map(int, part.split('-'))
                ports.extend(range(start, end + 1))
            else:
                ports.append(int(part))
        return ports

    def run_scan(self):
        print(f"[+] Scanning {self.args.target}...")
        self.target_ip = socket.gethostbyname(self.args.target)
        ports = self.parse_ports(self.args.ports)
        start_time = time.time()

        with ThreadPoolExecutor(max_workers=self.args.threads) as executor:
            executor.map(self.scan_port, ports)

        end_time = time.time()
        print(f"[+] Scan completed in {end_time - start_time:.2f} seconds")
        print(f"[+] Found {len(self.open_ports)} open ports: {sorted(self.open_ports)}")

    def run(self):
        print("=" * 60)
        print("           Simple Port Scanner")
        print("=" * 60)

        if not self.validate_target(self.args.target):
            print(f"[-] Invalid target: {self.args.target}")
            sys.exit(1)

        try:
            self.run_scan()
        except KeyboardInterrupt:
            print("\n[!] Scan interrupted")
            sys.exit(0)

if __name__ == "__main__":
    PortScanner()
```

## Phase 2: Testing and Validation

### Unit Testing Setup
```python
import pytest
from unittest.mock import Mock, patch, MagicMock
import sys
import os

sys.path.insert(0, os.path.dirname(os.path.dirname(__file__)))

from your_tool import SecurityTool

class TestSecurityTool:
    def setup_method(self):
        self.tool = SecurityTool.__new__(SecurityTool)

    def test_validate_target_valid_ip(self):
        assert self.tool.validate_target("192.168.1.1") == True

    def test_validate_target_invalid_ip(self):
        assert self.tool.validate_target("999.999.999.999") == False

    @patch('socket.gethostbyname')
    def test_validate_target_hostname(self, mock_gethostbyname):
        mock_gethostbyname.return_value = "192.168.1.1"
        assert self.tool.validate_target("example.com") == True

        mock_gethostbyname.side_effect = socket.gaierror
        assert self.tool.validate_target("invalid.domain") == False
```

### Integration Testing
```python
import subprocess
import sys

class TestCLIIntegration:
    def test_cli_help_output(self):
        result = subprocess.run(
            [sys.executable, 'your_tool.py', '--help'],
            capture_output=True, text=True
        )
        assert result.returncode == 0
        assert 'usage:' in result.stdout.lower()

    def test_cli_invalid_target(self):
        result = subprocess.run(
            [sys.executable, 'your_tool.py', '-t', 'invalid.target'],
            capture_output=True, text=True
        )
        assert result.returncode != 0
```

## Best Practices

1. **Input Validation**: Always validate user inputs
2. **Error Handling**: Use try-except for robustness
3. **Threading**: Use ThreadPoolExecutor for concurrency
4. **Timeouts**: Set timeouts to prevent hanging
5. **Logging**: Implement proper logging
6. **Resource Management**: Close connections properly
7. **User Feedback**: Provide clear output

## Security Considerations

- Only use for authorized security testing
- Implement rate limiting
- Handle sensitive data carefully
- Follow responsible disclosure

## Usage

To use this skill:
1. Specify the tool type (scanner/exploit/recon/custom)
2. Provide tool name and description
3. Follow the implementation phases
4. Add comprehensive testing
5. Document usage and security considerations