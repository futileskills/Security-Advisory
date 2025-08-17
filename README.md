CVE Request: Unauthenticated Command Injection in Epson POS Printers
Summary

This document details an unauthenticated command injection vulnerability in a range of Epson Point of Sale (POS) printers. The vulnerability allows an attacker to connect to the printer's network interface on a specific port and send raw, unauthenticated ESC/POS commands. Successful exploitation grants a remote attacker on the same network the ability to execute arbitrary commands, such as opening the cash drawer, without any form of authentication. This vulnerability is believed to affect numerous models using the same underlying protocol.
1. Vulnerability Details

    Vulnerability Type: Missing Authentication for Critical Function (CWE-306), Improper Access Control (CWE-284), Unauthenticated Command Injection (CWE-77: Improper Neutralization of Special Elements used in a Command).

    Vendor: Epson

    Affected Products: Epson POS printers, specifically identified on the TM-M30II (m362c) model. It is highly likely that other Epson and third-party printers using the same ESC/POS command set over a raw TCP/IP connection are also affected. This includes models in the TM-T and TM-M series.

    Affected Versions: All firmware versions prior to an as-yet-unreleased patch.

    Attack Vector: Remote, unauthenticated network connection.

    Protocol: Raw TCP/IP on port 9100.

2. Technical Description

Epson POS printers are designed to receive raw printer commands over a TCP connection on port 9100. These commands use a language known as ESC/POS, which is a de facto industry standard for thermal receipt printers. The printer's firmware does not implement any authentication or authorization mechanisms to validate the origin of these commands. As a result, any host on the same network segment can establish a connection and send commands to the printer.

The vulnerability is rooted in the printer's blind trust of commands received over this port. Since the ESC/POS command set includes instructions for critical hardware functions like opening the cash drawer (ESC p m t1 t2), an attacker can exploit this lack of authentication to trigger a physical action.

Crucial to the attack's success is the attacker's presence on the same network or VLAN as the printer. This vulnerability highlights the critical importance of proper network segmentation. If network isolation is breached, an attacker can leverage this vulnerability to gain control of the device.
3. Proof of Concept (PoC)

The following steps demonstrate the vulnerability using a standard command-line utility.

Step-by-step reproduction:

    Connect to the target printer's IP address on port 9100.

    Once the connection is established, send the following raw hex command directly to the printer.

printf '\x1b\x70\x00\x19\x32' | nc <PRINTER_IP_ADDRESS> 9100

Upon receipt of this hex sequence, the printer will immediately open its attached cash drawer.

Analysis of the command:

    \x1b (ESC): The escape character, which signals the start of a command sequence.

    \x70 (p): The command to activate a peripheral, in this case, the cash drawer.

    \x00: Selects the cash drawer.

    \x19: Sets the ON time for the pulse.

    \x32: Sets the OFF time for the pulse.

This specific command sequence is for opening the cash drawer, but an attacker could also send other commands, such as a paper cut command (\x1d\x56\x01\x40), to cause physical disruption without any credentials.
4. Impact

Exploitation of this vulnerability has significant consequences for businesses and organizations:

    Financial Loss: A remote attacker can open the cash drawer at will, enabling physical theft of cash and other valuables.

    Disruption of Business Operations: An attacker could maliciously trigger actions like printing endless receipts or cutting paper, causing a physical denial of service. The ability to print arbitrary text could be used for social engineering.

    Social Engineering and Information Gathering: The vulnerability allows an attacker to physically interact with the business environment, which can be used as a tool for social engineering or potentially printing sensitive information.

5. Suggested Mitigation

Vendor-side:

    Implement proper authentication and authorization for all commands sent over the network port.

    Consider an optional security mode that restricts access to port 9100 to a list of allowed IP addresses (IP filtering).

    Implement a secure printing protocol like IPPS (IPP over TLS) that supports authentication.

User-side:

    Isolate all POS devices on a private, segmented network that is not directly accessible from the main corporate or public Wi-Fi network.

    Configure firewall rules to explicitly deny any traffic to the printer's port 9100 that does not originate from the dedicated POS terminal or application server.# Security-Advisory-
Security Advisory for Epson printer/point of sale points
