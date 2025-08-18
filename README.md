# Security Advisory: Unauthenticated Command Injection in Epson POS Printers

 <p align="center">
      # Your Header Text Here
    </p>

This document details a critical unauthenticated command injection vulnerability affecting a range of Epson Point of Sale (POS) printers. The flaw allows a remote attacker to connect to a printer's network interface and send raw, unauthenticated ESC/POS commands. Successful exploitation grants an attacker on the same network the ability to execute arbitrary commands, such as opening the cash drawer, without any form of authentication. This vulnerability is believed to affect numerous models using the same underlying protocol.
**1. Vulnerability Details**

Vulnerability Type:

   &nbsp;&nbsp;&nbsp;&nbsp;Missing Authentication for Critical Function (CWE-306)

   &nbsp;&nbsp;&nbsp;&nbsp;Improper Access Control (CWE-284)

   &nbsp;&nbsp;&nbsp;&nbsp;Unauthenticated Command Injection (CWE-77: Improper Neutralization of Special Elements used in a Command)

**Vendor: Epson**

**Affected Products:** Epson POS printers, specifically identified on the TM-M30II (m362c) model. It is highly likely that other Epson and third-party printers using the same ESC/POS command set over a raw TCP/IP connection are also affected. This includes models in the TM-T and TM-M series.

&nbsp;&nbsp;&nbsp;&nbsp;Affected Versions: All firmware versions prior to an as-yet-unreleased patch.

&nbsp;&nbsp;&nbsp;&nbsp;Attack Vector: Remote, unauthenticated network connection.

&nbsp;&nbsp;&nbsp;&nbsp;Protocol: Raw TCP/IP on port 9100.

### 2. Technical Description

Epson POS printers are designed to receive raw printer commands over a TCP connection on port 9100. These commands use a language called ESC/POS, an industry standard for thermal receipt printers.

The printer's firmware does not implement any authentication or authorization mechanisms to validate the origin of these commands. Any host on the same network can establish a connection and send commands. This lack of authentication is a significant security vulnerability in modern, IP-networked environments.

Since the ESC/POS command set includes instructions for critical hardware functions like opening the cash drawer (ESC p m t1 t2), an attacker can exploit this design flaw to trigger physical actions.

The vulnerability highlights the critical importance of proper network segmentation. If isolation measures mandated by standards like PCI DSS are breached, an attacker can leverage this vulnerability to gain control of the device.

### 3. Proof of Concept (PoC)

The following steps demonstrate the vulnerability using a standard command-line utility.

Step-by-step reproduction:

   Connect to the target printer's IP address on port 9100.

   Once the connection is established, send the following raw hex command directly to the printer.

    ```
     printf '\x1b\x70\x00\x19\x32' | nc <PRINTER_IP_ADDRESS> 9100
    ```

Upon receipt of this hex sequence, the printer will immediately open its attached cash drawer.

Analysis of the command:

   \x1b (ESC): The escape character, signaling the start of a command sequence.

   \x70 (p): The command to activate a peripheral (the cash drawer).

   \x00: The first parameter, which selects the cash drawer connector pin 2.

   \x19: The second parameter, setting the ON pulse time.

   \x32: The third parameter, setting the OFF pulse time.

This command sequence specifically opens the cash drawer, but an attacker could also send other commands, such as a paper cut command (\x1d\x56\x01\x40), to cause physical disruption without any credentials.

### 4. Impact

Exploitation of this vulnerability has significant consequences for businesses and organizations:

   Financial Loss: A remote attacker can open the cash drawer at will, enabling physical theft.

   Disruption of Business Operations: An attacker could trigger actions like printing endless receipts or cutting paper, causing a physical denial of service. Arbitrary text could be printed to generate fraudulent receipts or malicious messages.

   Social Engineering: The vulnerability can be used as a tool for social engineering, creating distractions or printing information to trick employees or customers.

### 5. Suggested Mitigation

Vendor-side:

   Implement proper authentication and authorization for all commands sent over the network port.

   Consider adding an optional security mode, configurable via the device's web interface, that restricts access to port 9100 to a list of allowed IP addresses.

   Implement a secure printing protocol like IPPS (IPP over TLS) that supports authentication.

User-side:

   Isolate all POS devices on a private, segmented network that is not directly accessible from the main corporate or public Wi-Fi network.

   Configure firewall rules to explicitly deny any traffic to the printer's port 9100 that does not originate from the dedicated POS terminal or application server.
