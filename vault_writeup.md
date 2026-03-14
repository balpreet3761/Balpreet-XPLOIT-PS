# XPLOIT Vault Challenge - Writeup by balpreet3761

## 1. Initial Recon
- Ran `file chal` - ELF 64-bit PIE executable, not stripped
- Ran `strings chal` - found: ID_9921, ptrace usage, function names, vault strings
- Ran `objdump -d chal` - mapped all functions

## 2. Function Map
- security_watchdog: calls ptrace to detect debuggers, kills process if found
- user_authentication_module: checks Operator ID against value 0x3e7 (999)
- unlock_vault_sequence: computes unlock code via g_pid_seed XOR g_vault_byte XOR strlen(argv[0])
- check_vault_state: manages .vault_state file across runs
- omega_protocol_legacy: XOR decodes data with key 0xAB (decoy)
- compute_session_hash: FNV-1a hash (decoy)

## 3. Changes Made

### Patch 1 - Auth bypass at 0x198c
- Original: jne (0x75) skips admin path if variable != 999
- Problem: Variable initialized to 1, never set to 999 from user input
- Fix: Changed jne to je (0x74) - now enters admin path

### Patch 2 - ptrace bypass at 0x1918
- Original: jns (0x79) only skips kill if ptrace >= 0
- Problem: ptrace detects any analysis environment
- Fix: Changed to jmp (0xeb) - always skips kill path

### Patch 3 - Route to unlock_vault_sequence at 0x19d6
- Original: called exit(1) after admin accept
- Problem: unlock_vault_sequence never called
- Fix: NOPed 5 bytes, replaced exit call with call to unlock_vault_sequence (0x19f2)

### Patch 4 - Unlock code bypass at 0x1a61
- Original: jne exits if entered code doesnt match computed code
- Fix: Changed jne to je to accept input and proceed

## 4. Unlock Code Computation
From assembly at 0x1a0e-0x1a31:
unlock_code = g_pid_seed XOR g_vault_byte XOR strlen(argv[0])
- g_pid_seed: derived from PID XOR (PID >> 8) in emit_system_diagnostics
- g_vault_byte: single byte read from .vault_state file
- strlen(argv[0]): length of binary path string

## 5. Final Output
AUTHORIZATION ACCEPTED: Level 999 Admin
OMEGA_TOKEN active: XPLOIT-2026-OMEGA
Screenshot attached showing successful bypass.
