# Create Python CLI Tool for Exploit/Scanning

This phase guides the creation of a Python CLI tool with proper structure for security testing, exploit development, and scanning operations.

## Structure Requirements

### 1. OOP Design with Single Class
- Use object-oriented programming with one main class
- Class should handle initialization, argument parsing, and execution
- Follow the pattern: `__init__` → `run()` → specific methods

### 2. CLI Framework
Use `argparse` for command-line argument parsing:
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
        # Add your arguments here
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

### 3. Core Components

#### a. Target Validation
```python
def validate_target(self, target):
    """Validate target format and reachability"""
    import ipaddress
    import socket

    try:
        # Check if it's an IP address
        ipaddress.ip_address(target)
        return True
    except ValueError:
        try:
            # Check if it's a valid hostname
            socket.gethostbyname(target)
            return True
        except socket.gaierror:
            return False
```

#### b. Scanning Module
```python
def scan_target(self):
    """Perform security scanning"""
    if not self.validate_target(self.args.target):
        print(f"[-] Invalid target: {self.args.target}")
        return

    print(f"[+] Scanning target: {self.args.target}:{self.args.port}")

    # Implement your scanning logic here
    # - Port scanning
    # - Service detection
    # - Vulnerability checks
```

#### c. Exploit Module
```python
def exploit_target(self):
    """Execute exploit against target"""
    print(f"[+] Attempting exploit on {self.args.target}")

    # Implement exploit logic
    # - Payload generation
    # - Connection handling
    # - Exploit execution
    # - Result verification
```

### 4. Error Handling and Logging
```python
import logging
import traceback

class SecurityTool:
    def __init__(self):
        # Setup logging
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s'
        )
        self.logger = logging.getLogger(__name__)

        # Your existing code...

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

### 5. Output Formatting
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

### 6. Main Execution Flow
```python
def run(self):
    """Main execution method"""
    self.print_banner()

    if not self.args.target:
        self.parser.print_help()
        sys.exit(1)

    # Validate target
    if not self.validate_target(self.args.target):
        self.print_result(f"Invalid target: {self.args.target}", False)
        sys.exit(1)

    # Execute main functionality
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

## Example Implementation

Here's a complete example for a basic port scanner:

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
        self.parser = argparse.ArgumentParser(
            description="Simple Port Scanner",
            formatter_class=argparse.RawDescriptionHelpFormatter
        )
        self.setup_arguments()
        self.args = self.parser.parse_args()
        self.open_ports = []
        self.run()

    def setup_arguments(self):
        self.parser.add_argument(
            '-t', '--target',
            required=True,
            help='Target IP address or hostname'
        )
        self.parser.add_argument(
            '-p', '--ports',
            default='1-1024',
            help='Port range to scan (default: 1-1024)'
        )
        self.parser.add_argument(
            '--threads',
            type=int,
            default=100,
            help='Number of threads (default: 100)'
        )
        self.parser.add_argument(
            '--timeout',
            type=float,
            default=1.0,
            help='Connection timeout in seconds (default: 1.0)'
        )

    def validate_target(self, target):
        try:
            socket.gethostbyname(target)
            return True
        except socket.gaierror:
            return False

    def scan_port(self, port):
        """Scan a single port"""
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(self.args.timeout)
            result = sock.connect_ex((self.target_ip, port))
            sock.close()

            if result == 0:
                self.open_ports.append(port)
                print(f"[+] Port {port} is open")

        except Exception as e:
            pass

    def parse_ports(self, port_range):
        """Parse port range string like '1-1024' or '80,443,8080'"""
        ports = []
        for part in port_range.split(','):
            if '-' in part:
                start, end = map(int, part.split('-'))
                ports.extend(range(start, end + 1))
            else:
                ports.append(int(part))
        return ports

    def run_scan(self):
        """Execute the port scan"""
        print(f"[+] Scanning {self.args.target}...")
        print(f"[+] Port range: {self.args.ports}")
        print(f"[+] Threads: {self.args.threads}")
        print("-" * 50)

        self.target_ip = socket.gethostbyname(self.args.target)
        ports = self.parse_ports(self.args.ports)

        start_time = time.time()

        with ThreadPoolExecutor(max_workers=self.args.threads) as executor:
            executor.map(self.scan_port, ports)

        end_time = time.time()

        print("-" * 50)
        print(f"[+] Scan completed in {end_time - start_time:.2f} seconds")
        print(f"[+] Found {len(self.open_ports)} open ports: {sorted(self.open_ports)}")

    def run(self):
        """Main execution"""
        print("=" * 60)
        print("           Simple Port Scanner")
        print("           Author: HarshXor")
        print("=" * 60)

        if not self.validate_target(self.args.target):
            print(f"[-] Invalid target: {self.args.target}")
            sys.exit(1)

        try:
            self.run_scan()
        except KeyboardInterrupt:
            print("\n[!] Scan interrupted by user")
            sys.exit(0)
        except Exception as e:
            print(f"[-] Error: {str(e)}")
            sys.exit(1)

if __name__ == "__main__":
    PortScanner()
```

## Best Practices

1. **Input Validation**: Always validate user inputs
2. **Error Handling**: Use try-except blocks for robust operation
3. **Threading**: Use ThreadPoolExecutor for concurrent operations
4. **Timeouts**: Set appropriate timeouts to prevent hanging
5. **Logging**: Implement proper logging for debugging
6. **Resource Management**: Close sockets and file handles properly
7. **User Feedback**: Provide clear progress indicators and results

## Security Considerations

- Never run exploits against systems without permission
- Use these tools only for authorized security testing
- Implement rate limiting to avoid detection
- Handle sensitive data carefully
- Follow responsible disclosure practices