# Space Pirates 3
<img width="545" height="566" alt="image" src="https://github.com/user-attachments/assets/ed397186-6493-47db-90a1-a2d74d6f2891" />

**Mô tả:**
Challenge này cho ta 1 file .go flag được mã hóa qua nhiều lớp. Mục tiêu giải mã hóa để tìm flag.

### Phân tích
```go
/*
 * SPACE PIRATES CTF CHALLENGE - LEVEL 3: THE PIRATE KING'S VAULT
 * ===============================================================
 * You found the treasure, but wait... there's a note:
 * "This be but a fraction of me fortune! The REAL hoard lies in me secret vault,
 * protected by the most devious cipher ever created by pirate-kind. Only the 
 * cleverest of sea dogs can crack it. - Captain Blackbyte"
 * 
 * The Pirate King has combined the same 6 operations BUT with completely
 * different keys and parameters. This is the ultimate test!
 * 
 */

package main

import (
	"fmt"
	"os"
)

// The target encrypted vault combination (what we want the transformed input to become)
var target = [30]byte{
	0x60, 0x6D, 0x5D, 0x97, 0x2C, 0x04, 0xAF, 0x7C, 0xE2, 0x9E, 0x77, 0x85, 0xD1, 0x0F, 0x1D, 0x17, 0xD4, 0x30, 0xB7, 0x48, 0xDC, 0x48, 0x36, 0xC1, 0xCA, 0x28, 0xE1, 0x37, 0x58, 0x0F,
}

// The Pirate King's ULTIMATE XOR key (7 bytes - prime number for better mixing!)
var xorKey = [7]byte{0xC7, 0x2E, 0x89, 0x51, 0xB4, 0x6D, 0x1F}

// NEW: Rotation pattern (8 bytes, includes rotation by 0 which is identity)
var rotationPattern = [8]uint{7, 5, 3, 1, 6, 4, 2, 0}

// The Pirate King's subtraction constant (much larger than before!)
const magicSub byte = 0x93

// Chunk size for reversal (changed from 5 to 6!)
const chunkSize = 6

func printFlag(input string) {
	fmt.Println()
	fmt.Println("╔═══════════════════════════════════════════════════╗")
	fmt.Println("║                                                   ║")
	fmt.Println("║           ⚜️  THE PIRATE KING'S VAULT  ⚜️          ║")
	fmt.Println("║                                                   ║")
	fmt.Println("║              _.-^^^^`````````````^^-.             ║")
	fmt.Println("║         _.-`^`                       `^-._        ║")
	fmt.Println("║      ,-`     💎 💰 ⚜️  💰 💎 ⚜️  💰 💎     `-,     ║")
	fmt.Println("║     /         💰 ⚜️  💎 💰 💎 ⚜️  💰         \\    ║")
	fmt.Println("║    /          ⚜️  💎 💰 ⚜️  💰 💎 ⚜️          \\   ║")
	fmt.Println("║   |    💎 💰 ⚜️  THE ULTIMATE HOARD ⚜️  💰 💎   |  ║")
	fmt.Println("║    \\          ⚜️  💎 💰 ⚜️  💰 💎 ⚜️          /   ║")
	fmt.Println("║     \\         💰 ⚜️  💎 💰 💎 ⚜️  💰         /    ║")
	fmt.Println("║      `-,     💎 💰 ⚜️  💰 💎 ⚜️  💰 💎     ,-`     ║")
	fmt.Println("║         `-._                       _,-`          ║")
	fmt.Println("║             `--..___________..--`                ║")
	fmt.Println("║                                                   ║")
	fmt.Println("╚═══════════════════════════════════════════════════╝")
	fmt.Println()
	fmt.Println("🏴‍☠️ ⚔️  LEVEL 3 COMPLETE! MASTER OF THE SEVEN SEAS! ⚔️  🏴‍☠️")
	fmt.Println()
	fmt.Println("Ye've cracked the Pirate King's most devious cipher!")
	fmt.Println("The greatest treasure known to pirate-kind is yours!")
	fmt.Println()
	fmt.Println("Flag: ", input)
	fmt.Println()
	fmt.Println()
}

// rotateLeft rotates a byte left by n positions
// Bijection: ROL has inverse ROR
// Even rotation by 0 (identity) is bijective: ROL(x,0) = x, ROR(x,0) = x
func rotateLeft(b byte, n uint) byte {
	n = n % 8 // Ensure n is in range [0,7]
	return (b << n) | (b >> (8 - n))
}

// OPERATION 1: XOR with rotating key (7 bytes now for better coverage)
// Bijection: (x ⊕ k) ⊕ k = x
// Longer key doesn't affect bijectivity, just cycling pattern
func applyUltimateQuantumCipher(buffer []byte) {
	for i := range buffer {
		buffer[i] ^= xorKey[i%len(xorKey)]
	}
}

// OPERATION 2: Rotate Left with new pattern (includes rotation by 0)
// Bijection: ROL⁻¹ = ROR with same amount
// Rotation by 0 at position 7 is identity but still bijective
func applyStellarRotationV2(buffer []byte) {
	for i := range buffer {
		rotation := rotationPattern[i%len(rotationPattern)]
		buffer[i] = rotateLeft(buffer[i], rotation)
	}
}

// OPERATION 3: Swap adjacent byte pairs
// Bijection: Self-inverse, f(f(x)) = x
func applySpatialTransposition(buffer []byte) {
	for i := 0; i < len(buffer)-1; i += 2 {
		buffer[i], buffer[i+1] = buffer[i+1], buffer[i]
	}
}

// OPERATION 4: Subtract magic constant (NEW VALUE: 0x93)
// Bijection: (x - k) + k ≡ x (mod 256)
// Larger constant increases difficulty but not mathematical properties
func applyGravitationalShiftV3(buffer []byte) {
	for i := range buffer {
		buffer[i] -= magicSub
	}
}

// OPERATION 5: Reverse bytes in chunks of 6 (CHANGED from 5!)
// Splits 30 bytes into 5 chunks of 6 bytes each
// Bijection: Reversal is self-inverse, f(f(x)) = x
// 30 = 6 * 5, so all bytes are processed (no remainder)
func applyTemporalInversionV2(buffer []byte) {
	for chunkStart := 0; chunkStart < len(buffer); chunkStart += chunkSize {
		chunkEnd := chunkStart + chunkSize
		if chunkEnd > len(buffer) {
			chunkEnd = len(buffer)
		}
		// Reverse the chunk in place
		for i, j := chunkStart, chunkEnd-1; i < j; i, j = i+1, j-1 {
			buffer[i], buffer[j] = buffer[j], buffer[i]
		}
	}
}

// OPERATION 6: XOR each byte with (position² + position) mod 256
// Enhanced from Level 2's position²
// Bijection: (x ⊕ k) ⊕ k = x (XOR involution)
// The function (i² + i) mod 256 is deterministic and unique per position
func applyCoordinateCalibrationV3(buffer []byte) {
	for i := range buffer {
		// Calculate i² + i mod 256
		positionValue := ((i * i) + i) % 256
		buffer[i] ^= byte(positionValue)
	}
}

func processVault(input string) ([30]byte, error) {
	var result [30]byte

	// Validate length
	if len(input) != 30 {
		return result, fmt.Errorf(
			"Invalid vault combination",
		)
	}

	// Convert to byte array
	copy(result[:], input)
	buffer := result[:] // Create slice for easier manipulation

	fmt.Println("Decrypting the Pirate King's vault...\n")

	fmt.Println("[1/6] Applying ultimate quantum entanglement cipher...")
	applyUltimateQuantumCipher(buffer)

	fmt.Println("[2/6] Applying advanced stellar rotation...")
	applyStellarRotationV2(buffer)

	fmt.Println("[3/6] Applying spatial transposition...")
	applySpatialTransposition(buffer)

	fmt.Println("[4/6] Applying extreme gravitational shift...")
	applyGravitationalShiftV3(buffer)

	fmt.Println("[5/6] Applying enhanced temporal inversion...")
	applyTemporalInversionV2(buffer)

	fmt.Println("[6/6] Applying master coordinate calibration...")
	applyCoordinateCalibrationV3(buffer)

	return result, nil
}

func main() {
	fmt.Println("╔════════════════════════════════════════════════════╗")
	fmt.Println("║   PIRATE KING'S VAULT DECODER v3.0 - MASTER LEVEL  ║")
	fmt.Println("║                LEVEL 3: ULTIMATE                   ║")
	fmt.Println("╚════════════════════════════════════════════════════╝")
	fmt.Println()
	fmt.Println()
	fmt.Println("You've found the treasure, but Captain Blackbyte")
	fmt.Println("left a note: 'Me REAL fortune lies in me secret vault!'")
	fmt.Println()

	// Check arguments
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "  Usage: %s <vault_combination>\n", os.Args[0])
		os.Exit(1)
	}

	input := os.Args[1]

	// Process the vault combination
	result, err := processVault(input)
	if err != nil {
		fmt.Fprintf(os.Stderr, " %v\n", err)
		os.Exit(1)
	}

	fmt.Println("\nVerifying vault combination against the master lock...")

	// Check if result matches target
	if result == target {
		printFlag(input)
	} else {
		fmt.Println("\nVAULT REMAINS SEALED!\n")
		fmt.Println("The combination doesn't match the Pirate King's lock.")

		// Debug output
		// fmt.Print("Your result (hex): ")
		// for _, b := range result {
		// 	fmt.Printf("0x%02X, ", b)
		// }
		fmt.Println("\n")
		os.Exit(1)
	}
}
```
- Chương trình thực hiện 6 hành động:
+ Hành động 1: XOR với xorKey
+ Hành động 2: Rotate Left
+ Hành động 3: Swap cặp byte
+ Hành động 4: Subtract magicSub (0x93)
+ Hành động 5: Reverse theo chunk 6 byte
+ Hành động 6: XOR với (i² + i) % 256
- Ta chỉ cần làm ngược lại các hành động trên là có thể lấy được flag.

### Script

```py
# SPACE PIRATES CTF – LEVEL 3 SOLVER
# ----------------------------------

target = [0x60,0x6D,0x5D,0x97,0x2C,0x04,0xAF,0x7C,0xE2,0x9E,0x77,0x85,
          0xD1,0x0F,0x1D,0x17,0xD4,0x30,0xB7,0x48,0xDC,0x48,0x36,0xC1,
          0xCA,0x28,0xE1,0x37,0x58,0x0F]

xorKey = [0xC7,0x2E,0x89,0x51,0xB4,0x6D,0x1F]
rotationPattern = [7,5,3,1,6,4,2,0]
magicSub = 0x93
chunkSize = 6

def ror(b, n):
    n %= 8
    return ((b >> n) | ((b << (8 - n)) & 0xFF)) & 0xFF

def inverse_pipeline(buf):
    buf = buf.copy()

    # Step 6 inverse: XOR (i² + i)
    for i in range(len(buf)):
        posVal = ((i * i) + i) % 256
        buf[i] ^= posVal

    # Step 5 inverse: reverse chunks of 6
    for start in range(0, len(buf), chunkSize):
        end = start + chunkSize
        buf[start:end] = reversed(buf[start:end])

    # Step 4 inverse: add magicSub
    buf = [(b + magicSub) & 0xFF for b in buf]

    # Step 3 inverse: swap pairs
    for i in range(0, len(buf) - 1, 2):
        buf[i], buf[i+1] = buf[i+1], buf[i]

    # Step 2 inverse: ROR with rotationPattern
    for i in range(len(buf)):
        buf[i] = ror(buf[i], rotationPattern[i % len(rotationPattern)])

    # Step 1 inverse: XOR with key
    for i in range(len(buf)):
        buf[i] ^= xorKey[i % len(xorKey)]

    return buf

result = inverse_pipeline(target)

flag = bytes(result)
print("Flag:", flag.decode())

```
Flag: PCTF{M4ST3R_0F_TH3_S3V3N_S34S}
