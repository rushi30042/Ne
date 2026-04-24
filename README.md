# SEC-251-CS-P | Networking Lab — All Slip Programs
### S.Y.B.Sc Computer Science | Savitribai Phule Pune University | 2025-26

---

## ⚡ CRC Quick Reference
| Data | Divisor | Remainder | Transmitted Frame | Slips |
|------|---------|-----------|-------------------|-------|
| 101010111 | 1011 | 100 | 101010111**100** | 1,10,15,20 |
| 101000 | 1011 | 110 | 101000**110** | 7 |
| 100100 | 1101 | 001 | 100100**001** | 9,12,17,19 |

---

## SLIP 1

### Q2B — Character Stuffing & Bit Stuffing

```c
#include <stdio.h>
#include <string.h>

/* Character Stuffing: 'F' = flag, 'E' = escape character */
void charStuff(char *data) {
    char result[100] = "";
    int j = 0;
    result[j++] = 'F';                    // Opening flag
    for (int i = 0; data[i] != '\0'; i++) {
        if (data[i] == 'F' || data[i] == 'E')
            result[j++] = 'E';            // Insert escape before special char
        result[j++] = data[i];
    }
    result[j++] = 'F';                    // Closing flag
    result[j] = '\0';
    printf("Original  : %s\n", data);
    printf("Stuffed   : %s\n", result);
}

/* Bit Stuffing: insert 0 after every five consecutive 1s */
void bitStuff(int bits[], int n) {
    int result[100], k = 0, count = 0;
    printf("\nOriginal Bits  : ");
    for (int i = 0; i < n; i++) printf("%d", bits[i]);
    printf("\n");
    for (int i = 0; i < n; i++) {
        result[k++] = bits[i];
        if (bits[i] == 1) {
            count++;
            if (count == 5) { result[k++] = 0; count = 0; }
        } else count = 0;
    }
    printf("After Stuffing : ");
    for (int i = 0; i < k; i++) printf("%d", result[i]);
    printf("\n");
}

int main() {
    printf("=== CHARACTER STUFFING ===\n");
    char data[] = "HFEllo";
    charStuff(data);
    printf("\n=== BIT STUFFING ===\n");
    int bits[] = {1, 1, 1, 1, 1, 0, 1};
    bitStuff(bits, 7);
    return 0;
}
```

### Q2B OR — Password Strength Checker

```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>

int main() {
    char password[50];
    int hasUpper = 0, hasLower = 0, hasDigit = 0, hasSpecial = 0;
    printf("Enter password: ");
    scanf("%s", password);
    for (int i = 0; password[i] != '\0'; i++) {
        if      (isupper(password[i])) hasUpper   = 1;
        else if (islower(password[i])) hasLower   = 1;
        else if (isdigit(password[i])) hasDigit   = 1;
        else                           hasSpecial = 1;
    }
    int score = hasUpper + hasLower + hasDigit + hasSpecial;
    int len   = strlen(password);
    printf("\nLength: %d | Score: %d/4\n", len, score);
    if      (len >= 8 && score == 4) printf("Strength: STRONG\n");
    else if (len >= 6 && score >= 2) printf("Strength: MEDIUM\n");
    else                             printf("Strength: WEAK\n");
    return 0;
}
```

---

## SLIP 2

### Q2B — Hash Table Simulation (Linear Probing)

```c
#include <stdio.h>
#include <string.h>
#define SIZE 7

int  keys[SIZE];
char values[SIZE][20];

void initTable() {
    for (int i = 0; i < SIZE; i++) { keys[i] = -1; values[i][0] = '\0'; }
}

void insert(int key, char *val) {
    int index = key % SIZE;
    while (keys[index] != -1) index = (index + 1) % SIZE;
    keys[index] = key;
    strcpy(values[index], val);
    printf("Inserted key=%d (\"%s\") at index %d\n", key, val, index);
}

void search(int key) {
    int index = key % SIZE, start = index;
    while (keys[index] != -1) {
        if (keys[index] == key) {
            printf("Found: key=%d => \"%s\" at index %d\n", key, values[index], index);
            return;
        }
        index = (index + 1) % SIZE;
        if (index == start) break;
    }
    printf("Key %d NOT FOUND.\n", key);
}

int main() {
    initTable();
    insert(10, "Alice");
    insert(20, "Bob");
    insert(17, "Charlie");
    insert(14, "David");
    search(20);
    search(17);
    search(99);
    return 0;
}
```

### Q2B OR — Character Count Framing *(Also: Slip 11)*

```c
#include <stdio.h>
#include <string.h>

int main() {
    char frames[3][20] = {"HELLO", "NET", "WORK"};
    int  numFrames = 3;
    printf("=== CHARACTER COUNT FRAMING ===\n\n");
    printf("Framed Output:\n");
    for (int i = 0; i < numFrames; i++) {
        int frameLen = (int)strlen(frames[i]) + 1;  // +1 for count byte itself
        printf("[%d]%s ", frameLen, frames[i]);
    }
    printf("\n\nExplanation:\n");
    for (int i = 0; i < numFrames; i++) {
        int frameLen = (int)strlen(frames[i]) + 1;
        printf("  Frame %d: length=%d, data=\"%s\"\n", i+1, frameLen, frames[i]);
    }
    return 0;
}
```

---

## SLIP 3

### Q2B — Bus Topology Simulation

```c
#include <stdio.h>

int main() {
    int n;
    printf("Enter number of nodes: ");
    scanf("%d", &n);
    printf("\n========== BUS TOPOLOGY ==========\n\n");
    for (int i = 1; i <= n; i++) {
        printf("  [PC%d]", i);
        if (i < n) printf("---");
    }
    printf("\n        MAIN BUS CABLE\n\n");
    printf("Nodes: %d | Single shared communication line\n", n);
    return 0;
}
```

### Q2B OR — CRC Error Detection *(Also: Slips 6, 16, 20)*

```c
#include <stdio.h>

int data[20], divisor[20], remainder[20];
int dataLen, divLen;

void computeCRC() {
    int temp[40];
    int tempLen = dataLen + divLen - 1;
    for (int i = 0; i < dataLen; i++) temp[i] = data[i];
    for (int i = dataLen; i < tempLen; i++) temp[i] = 0;  // Append zeros
    for (int i = 0; i < dataLen; i++)
        if (temp[i] == 1)
            for (int j = 0; j < divLen; j++) temp[i+j] ^= divisor[j];
    for (int i = 0; i < divLen - 1; i++) remainder[i] = temp[dataLen + i];
}

int main() {
    int bit;
    printf("Enter data bits (-1 to stop): ");
    dataLen = 0;
    while (scanf("%d", &bit) && bit != -1) data[dataLen++] = bit;
    printf("Enter divisor bits (-1 to stop): ");
    divLen = 0;
    while (scanf("%d", &bit) && bit != -1) divisor[divLen++] = bit;
    computeCRC();
    printf("\nCRC Remainder    : ");
    for (int i = 0; i < divLen-1; i++) printf("%d", remainder[i]);
    printf("\nTransmitted Frame: ");
    for (int i = 0; i < dataLen; i++) printf("%d", data[i]);
    for (int i = 0; i < divLen-1; i++) printf("%d", remainder[i]);
    printf("\n");
    return 0;
}
```

---

## SLIP 4

### Q2B — Phishing Pattern Detection

```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>

int main() {
    char email[500], lower[500];
    printf("Enter email text:\n");
    fgets(email, 500, stdin);
    strcpy(lower, email);
    for (int i = 0; lower[i]; i++) lower[i] = tolower(lower[i]);
    char *keywords[] = {"click here", "verify", "urgent", "password", "free prize"};
    int found = 0;
    printf("\n=== PHISHING ANALYSIS ===\n");
    for (int i = 0; i < 5; i++)
        if (strstr(lower, keywords[i])) {
            printf("  [!] Keyword found: \"%s\"\n", keywords[i]);
            found++;
        }
    printf("\nMatched: %d/5\n", found);
    if      (found > 2) printf("Verdict: PHISHING EMAIL\n");
    else if (found > 0) printf("Verdict: SUSPICIOUS\n");
    else                printf("Verdict: SAFE\n");
    return 0;
}
```

### Q2B OR — Check Private or Public IP Address

```c
#include <stdio.h>

/* RFC 1918 Private Ranges:
   Class A: 10.0.0.0    - 10.255.255.255
   Class B: 172.16.0.0  - 172.31.255.255
   Class C: 192.168.0.0 - 192.168.255.255 */

int main() {
    int a, b, c, d;
    printf("Enter IP address (e.g. 192 168 1 1): ");
    scanf("%d %d %d %d", &a, &b, &c, &d);
    printf("\nIP Address: %d.%d.%d.%d\n", a, b, c, d);
    if      (a == 10)
        printf("Type: PRIVATE IP (Class A)\n");
    else if (a == 172 && b >= 16 && b <= 31)
        printf("Type: PRIVATE IP (Class B)\n");
    else if (a == 192 && b == 168)
        printf("Type: PRIVATE IP (Class C)\n");
    else if (a == 127)
        printf("Type: LOOPBACK ADDRESS\n");
    else
        printf("Type: PUBLIC IP\n");
    return 0;
}
```

---

## SLIP 5

### Q2B — Ring Topology Simulation

```c
#include <stdio.h>

int main() {
    int n;
    printf("Enter number of nodes: ");
    scanf("%d", &n);
    printf("\n========== RING TOPOLOGY ==========\n\n");
    printf("         [PC1]\n");
    printf("        /     \\\n");
    for (int i = 2; i <= n/2; i++)
        printf("   [PC%-2d]       [PC%d]\n", i, n-i+2);
    printf("        \\     /\n");
    printf("         [PC%d]\n\n", n/2+1);
    printf("Circular Connections:\n");
    for (int i = 1; i <= n; i++)
        printf("  PC%d <--> PC%d\n", i, (i % n) + 1);
    printf("\nTotal Nodes: %d | Total Links: %d\n", n, n);
    return 0;
}
```

### Q2B OR — VLAN1 Configuration using Structure *(Also: Slip 15)*

```c
#include <stdio.h>
#include <string.h>

struct VLAN {
    int  id;
    char ip[20];
    char mask[20];
    char gateway[20];
};

int main() {
    struct VLAN v;
    printf("Enter VLAN ID    : "); scanf("%d",  &v.id);
    printf("Enter IP Address : "); scanf("%s",   v.ip);
    printf("Enter Subnet Mask: "); scanf("%s",   v.mask);
    printf("Enter Gateway    : "); scanf("%s",   v.gateway);
    printf("\n=== Generated Cisco IOS Commands ===\n\n");
    printf("interface Vlan%d\n",      v.id);
    printf(" ip address %s %s\n",     v.ip, v.mask);
    printf(" no shutdown\n!\n");
    printf("ip default-gateway %s\n", v.gateway);
    return 0;
}
```

---

## SLIP 6

### Q2B — CRC Method
> ★ Same program as **Slip 3 OR** (CRC Error Detection) — refer above.

### Q2B OR — Star Topology Simulation *(Also: Slip 12)*

```c
#include <stdio.h>

int main() {
    int n;
    printf("Enter number of PCs: ");
    scanf("%d", &n);
    printf("\n========== STAR TOPOLOGY ==========\n\n");
    printf("       [SWITCH]\n");
    printf("      /   |    \\\n");
    for (int i = 1; i <= n; i++) {
        printf("  [PC%d]", i);
        if (i % 3 == 0 || i == n) printf("\n");
    }
    printf("\nConnections:\n");
    for (int i = 1; i <= n; i++)
        printf("  PC%d <--> SWITCH\n", i);
    printf("\nTotal PCs: %d | Total Links: %d\n", n, n);
    return 0;
}
```

---

## SLIP 7

### Q1 OR — CRC Manual Calculation
```
Data = 101000, Divisor = 1011
Append 3 zeros : 101000000
XOR Division   : Remainder = 110
Transmitted Frame = 101000110
```

### Q2B — Mesh Topology Simulation *(Also: Slips 15, 20)*

```c
#include <stdio.h>

int main() {
    int n;
    printf("Enter number of nodes: ");
    scanf("%d", &n);
    int links = n * (n - 1) / 2;
    printf("\n========== MESH TOPOLOGY ==========\n");
    printf("Nodes : %d\n", n);
    printf("Links : %d  [n*(n-1)/2]\n\n", links);
    printf("All Connections:\n");
    for (int i = 1; i <= n; i++)
        for (int j = i+1; j <= n; j++)
            printf("  PC%d <--> PC%d\n", i, j);
    return 0;
}
```

### Q2B OR — Menu-Based Switch Configuration Simulator *(Also: Slips 11, 12)*

```c
#include <stdio.h>
#include <string.h>

char hostname[30] = "Switch";
char password[20] = "";
char ipAddr[20]   = "";
char subnetMask[20] = "";

void showConfig() {
    printf("\n--- Running Configuration ---\n");
    printf("hostname %s\n", hostname);
    if (strlen(password) > 0) printf("enable secret %s\n", password);
    printf("!\ninterface Vlan1\n");
    if (strlen(ipAddr) > 0)
        printf(" ip address %s %s\n", ipAddr, subnetMask);
    printf(" no shutdown\n!\nend\n");
    printf("-----------------------------\n");
}

int main() {
    int choice;
    printf("=== Cisco Switch Configuration Simulator ===\n");
    do {
        printf("\n1.Set Hostname  2.Set Password  3.Set IP  4.Show Config  5.Exit\n");
        printf("Choice: "); scanf("%d", &choice);
        switch(choice) {
            case 1: printf("Hostname: "); scanf("%s", hostname); break;
            case 2: printf("Password: "); scanf("%s", password); break;
            case 3:
                printf("IP Address : "); scanf("%s", ipAddr);
                printf("Subnet Mask: "); scanf("%s", subnetMask);
                break;
            case 4: showConfig(); break;
            case 5: printf("Exiting...\n"); break;
            default: printf("Invalid choice!\n");
        }
    } while (choice != 5);
    return 0;
}
```

---

## SLIP 8

### Q2B — show running-config & ping Simulation *(Also: Slip 17)*

```c
#include <stdio.h>
#include <string.h>

char runningConfig[] =
    "hostname Router\n"
    "enable secret cisco\n!\n"
    "interface GigabitEthernet0/0\n"
    " ip address 192.168.1.1 255.255.255.0\n"
    " no shutdown\n!\nend\n";

void showRunningConfig() {
    printf("Router# show running-config\n");
    printf("Building configuration...\n\n%s", runningConfig);
}

void ping(char *ip) {
    printf("\nRouter# ping %s\n", ip);
    printf("Sending 5 ICMP Echos to %s:\n\n", ip);
    for (int i = 1; i <= 5; i++)
        printf("  Reply from %s: TTL=64 time=%dms\n", ip, i%2+1);
    printf("\nPackets: Sent=5, Received=5, Lost=0 (0%% loss)\n");
}

int main() {
    showRunningConfig();
    ping("192.168.1.1");
    return 0;
}
```

### Q2B OR — Dynamic Routing Simulation (RIP)

```c
#include <stdio.h>

struct Route {
    char network[20];
    int  hopCount;
    char via[30];
};

int main() {
    struct Route rt[] = {
        {"192.168.1.0", 1, "Direct (connected)"},
        {"10.0.0.0",    2, "192.168.1.2"},
        {"172.16.0.0",  3, "192.168.1.2"}
    };
    printf("Router# show ip route rip\n\nCodes: R-RIP, C-Connected\n\n");
    printf("%-20s %-12s %s\n", "Network", "Hop Count", "Via");
    printf("%-20s %-12s %s\n", "----------", "----------", "---");
    for (int i = 0; i < 3; i++) {
        char code = (rt[i].hopCount == 1) ? 'C' : 'R';
        printf("[%c] %-18s %-12d %s\n", code, rt[i].network, rt[i].hopCount, rt[i].via);
    }
    return 0;
}
```

---

## SLIP 9

### Q1 OR — CRC Manual Calculation
```
Data = 100100, Divisor = 1101
Append 3 zeros : 100100000
XOR Division   : Remainder = 001
Transmitted Frame = 100100001
```

### Q2B — Determine IP Address Class

```c
#include <stdio.h>

/* Class A: 1-126 | Class B: 128-191 | Class C: 192-223
   Class D: 224-239 (Multicast) | Class E: 240-255 (Reserved) */

int main() {
    int a, b, c, d;
    printf("Enter IP (a b c d): ");
    scanf("%d %d %d %d", &a, &b, &c, &d);
    printf("\nIP: %d.%d.%d.%d => ", a, b, c, d);
    if      (a >= 1   && a <= 126) printf("Class A | Mask: 255.0.0.0\n");
    else if (a == 127)             printf("Loopback\n");
    else if (a >= 128 && a <= 191) printf("Class B | Mask: 255.255.0.0\n");
    else if (a >= 192 && a <= 223) printf("Class C | Mask: 255.255.255.0\n");
    else if (a >= 224 && a <= 239) printf("Class D (Multicast)\n");
    else                           printf("Class E (Reserved)\n");
    return 0;
}
```

### Q2B OR — Static Routing Table Simulation *(Also: Slip 14)*

```c
#include <stdio.h>

struct Route {
    char dest[20], mask[20], gw[20];
    int  metric;
};

int main() {
    struct Route rt[] = {
        {"192.168.2.0", "255.255.255.0", "192.168.1.2", 1},
        {"10.0.0.0",    "255.0.0.0",     "192.168.1.3", 2},
        {"0.0.0.0",     "0.0.0.0",       "192.168.1.1", 0}   // Default route
    };
    printf("Router# show ip route static\n\n");
    printf("%-18s %-18s %-18s %s\n", "Destination", "Mask", "Gateway", "Metric");
    printf("%-18s %-18s %-18s %s\n", "-----------", "----", "-------", "------");
    for (int i = 0; i < 3; i++) {
        char code = (rt[i].metric == 0) ? '*' : 'S';
        printf("[%c] %-16s %-18s %-18s %d\n",
            code, rt[i].dest, rt[i].mask, rt[i].gw, rt[i].metric);
    }
    return 0;
}
```

---

## SLIP 10

### Q1 OR — CRC Manual Calculation
```
Data = 101010111, Divisor = 1011
Append 3 zeros : 101010111000
XOR Division   : Remainder = 100
Transmitted Frame = 101010111100
```

### Q2B — Even Parity Error Detection

```c
#include <stdio.h>

/* Even Parity: total 1s in frame (including parity bit) must be even */

int main() {
    int bits[8], onesCount = 0;
    printf("Enter 7 data bits (space-separated): ");
    for (int i = 0; i < 7; i++) {
        scanf("%d", &bits[i]);
        onesCount += bits[i];
    }
    bits[7] = (onesCount % 2 == 0) ? 0 : 1;   // Parity bit
    printf("\nData bits (7)     : ");
    for (int i = 0; i < 7; i++) printf("%d ", bits[i]);
    printf("\nParity bit        : %d\n", bits[7]);
    printf("Transmitted frame : ");
    for (int i = 0; i < 8; i++) printf("%d", bits[i]);
    printf("\n");
    return 0;
}
```

### Q2B OR — Save Switch Config Output to Text File *(Also: Slip 14)*

```c
#include <stdio.h>

int main() {
    FILE *fp = fopen("config.txt", "w");
    if (fp == NULL) { printf("Error opening file!\n"); return 1; }
    fprintf(fp, "hostname Switch\n");
    fprintf(fp, "enable secret cisco\n!\n");
    fprintf(fp, "interface Vlan1\n");
    fprintf(fp, " ip address 192.168.1.10 255.255.255.0\n");
    fprintf(fp, " no shutdown\n!\n");
    fprintf(fp, "ip default-gateway 192.168.1.1\n!\nend\n");
    fclose(fp);
    printf("Config saved to config.txt\n");
    return 0;
}
```

---

## SLIP 11
> **Q2B** → Same as **Slip 2 OR** (Character Count Framing)
>
> **Q2B OR** → Same as **Slip 7 OR** (Menu-Based Switch Config)

---

## SLIP 12
> **Q2B** → Same as **Slip 6 OR** (Star Topology)
>
> **Q2B OR** → Same as **Slip 7 OR** (Menu-Based Switch Config)

---

## SLIP 13

### Q2B — Hybrid Topology Simulation

```c
#include <stdio.h>

int main() {
    printf("========== HYBRID TOPOLOGY ==========\n\n");
    printf("Star Group 1 (Switch 1):\n");
    printf("  PC1 ---|\n  PC2 ---[SW1]\n         |\n\n");
    printf("Star Group 2 (Switch 2):\n");
    printf("  PC3 ---|\n  PC4 ---[SW2]\n         |\n\n");
    printf("Bus Backbone:\n");
    printf("  [SW1]---[SW2]---[SW3]\n\n");
    printf("Ring Core:\n");
    printf("  [SW3] <--> [Router] <--> [SW1]\n\n");
    printf("Hybrid = Star + Bus + Ring combined.\n");
    return 0;
}
```

### Q2B OR — Save Full Switch Configuration to File

```c
#include <stdio.h>

int main() {
    FILE *fp = fopen("switch_config.txt", "w");
    if (fp == NULL) { printf("Error!\n"); return 1; }
    fprintf(fp, "enable\nconfigure terminal\n!\n");
    fprintf(fp, "hostname MySwitch\n");
    fprintf(fp, "enable secret cisco\n!\n");
    fprintf(fp, "interface Vlan1\n");
    fprintf(fp, " ip address 192.168.1.1 255.255.255.0\n no shutdown\n!\n");
    fprintf(fp, "ip default-gateway 192.168.1.254\n!\n");
    fprintf(fp, "line vty 0 4\n password telnet\n login\n!\nend\n");
    fclose(fp);
    printf("Saved to switch_config.txt\n");
    return 0;
}
```

---

## SLIP 14
> **Q2B** → Same as **Slip 9 OR** (Static Routing)
>
> **Q2B OR** → Same as **Slip 10 OR** (Save Config to File)

---

## SLIP 15
> **Q2B** → Same as **Slip 5 OR** (VLAN Config using Structure)
>
> **Q2B OR** → Same as **Slip 7 Q2B** (Mesh Topology)

---

## SLIP 16

### Q2B — Caesar Cipher – Encrypt & Decrypt *(Also: Slip 17 OR)*

```c
#include <stdio.h>
#include <string.h>

/* Encryption: C = (P + key) mod 26
   Decryption: P = (C - key + 26) mod 26 */

void encrypt(char *msg, int key, char *result) {
    for (int i = 0; msg[i]; i++) {
        if      (msg[i] >= 'a' && msg[i] <= 'z')
            result[i] = (char)(((msg[i]-'a'+key)%26)+'a');
        else if (msg[i] >= 'A' && msg[i] <= 'Z')
            result[i] = (char)(((msg[i]-'A'+key)%26)+'A');
        else
            result[i] = msg[i];
        result[i+1] = '\0';
    }
}

void decrypt(char *cipher, int key, char *result) {
    for (int i = 0; cipher[i]; i++) {
        if      (cipher[i] >= 'a' && cipher[i] <= 'z')
            result[i] = (char)(((cipher[i]-'a'-key+26)%26)+'a');
        else if (cipher[i] >= 'A' && cipher[i] <= 'Z')
            result[i] = (char)(((cipher[i]-'A'-key+26)%26)+'A');
        else
            result[i] = cipher[i];
        result[i+1] = '\0';
    }
}

int main() {
    char message[100], encrypted[100], decrypted[100];
    int  key;
    printf("=== CAESAR CIPHER ===\n\n");
    printf("Enter message : ");
    fgets(message, 100, stdin);
    int len = strlen(message);
    if (message[len-1] == '\n') message[len-1] = '\0';
    printf("Enter key     : "); scanf("%d", &key);
    key = key % 26;
    encrypt(message, key, encrypted);
    decrypt(encrypted, key, decrypted);
    printf("\nOriginal  : %s\n", message);
    printf("Encrypted : %s\n", encrypted);
    printf("Decrypted : %s\n", decrypted);
    return 0;
}
```

### Q2B OR — CRC Method
> ★ Same as **Slip 3 OR** (CRC Error Detection) — refer above.

---

## SLIP 17
> **Q2B** → Same as **Slip 8 Q2B** (show running-config & ping)
>
> **Q2B OR** → Same as **Slip 16 Q2B** (Caesar Cipher)

---

## SLIP 18

### Q2B — Phishing Email Simulation *(Also: Slip 19)*

```c
#include <stdio.h>

int main() {
    printf("========================================\n");
    printf("     PHISHING EMAIL SIMULATION\n");
    printf("========================================\n\n");
    printf("From    : support@paypa1.com\n");
    printf("Subject : URGENT - Verify your account NOW!\n\n");
    printf("Dear User,\n");
    printf("Your account will be SUSPENDED within 24 hours!\n");
    printf("Click here: http://fake-bank.com/login\n");
    printf("Enter your password immediately.\n\n");
    printf("========================================\n");
    printf("          RED FLAGS DETECTED\n");
    printf("========================================\n");
    printf("  [1] Fake domain: paypa1.com (not paypal.com)\n");
    printf("  [2] Urgent language: SUSPENDED, NOW!\n");
    printf("  [3] Suspicious link: fake-bank.com\n");
    printf("  [4] Requesting password via email\n");
    printf("========================================\n");
    printf("VERDICT: THIS IS A PHISHING EMAIL!\n");
    return 0;
}
```

### Q2B OR — NAT Simulation

```c
#include <stdio.h>
#include <string.h>

struct NAT {
    char privateIP[20];
    char publicIP[20];
    int  port;
};

int main() {
    struct NAT t[] = {
        {"192.168.1.10", "203.0.113.1", 1024},
        {"192.168.1.11", "203.0.113.1", 1025},
        {"192.168.1.12", "203.0.113.1", 1026}
    };
    printf("== NAT Table ==\n\n");
    printf("%-16s %-16s Port\n", "Private IP", "Public IP");
    printf("%-16s %-16s ----\n", "----------", "---------");
    for (int i = 0; i < 3; i++)
        printf("%-16s %-16s %d\n", t[i].privateIP, t[i].publicIP, t[i].port);
    return 0;
}
```

---

## SLIP 19

### Q2B — Phishing Simulation
> ★ Same as **Slip 18 Q2B** — refer above.

### Q2B OR — Verify NAT Translation

```c
#include <stdio.h>

int main() {
    printf("Router# show ip nat translations\n\n");
    printf("%-8s %-22s %-22s\n", "Proto", "Inside local", "Inside global");
    printf("%-8s %-22s %-22s\n", "-----", "------------", "-------------");
    printf("%-8s %-22s %-22s\n", "tcp", "192.168.1.10:1024", "203.0.113.1:1024");
    printf("%-8s %-22s %-22s\n", "tcp", "192.168.1.11:1025", "203.0.113.1:1025");
    printf("%-8s %-22s %-22s\n", "udp", "192.168.1.12:1026", "203.0.113.1:1026");
    printf("\nNAT translation verified. Total entries: 3\n");
    return 0;
}
```

---

## SLIP 20
> **Q2B** → Same as **Slip 7 Q2B** (Mesh Topology)
>
> **Q2B OR** → Same as **Slip 3 OR** (CRC Error Detection)

---

## How to Compile & Run
```bash
gcc filename.c -o output
./output
```

---

*SEC-251-CS-P | Networking Lab | S.Y.B.Sc CS | SPPU 2025-26*
