## Strategy

### Overview

Need to remove PCR duplicates from a SAM file and keep only one copy of each read

The following pieces of data will determine if we consider two reads duplicates:
- Chromosome - RNAME
- Strand - bit 16 in the bitwise flag (1 means reverse strand)
	- If bit 4 is true it means the read didn't map to the reference and should be discarded
- Start position (should include soft clipped regions, and if reversed the start position should be on the right)
	- POS column has leftmost position
	- Soft clipped regions are marked with S in the CIGAR string
- UMI - Last part of QNAME


### Pseudocode

```
read UMI file, create a list of the UMIs

reads = []

for each line in SAM file:
	separate line by tabs to access each column
	flag = FLAG
	if flag & 4:
		continue

	reversed = flag & 16
	chr = RNAME
	umi = extract UMI from QNAME
	cigar = CIGAR

	length = calculate length by adding numbers in CIGAR string

	left_start_pos = POS

	left_clipped = number of soft-clipped bases on the left (0 if there is no S on the left of the CIGAR string)

	start = left_start_pos - left_clipped + (length - 1 if reversed else 0)

	key = concatenate reversed, chr, umi, and start into a string
	if key in reads:
		continue
	else:
		add key to reads
		write line to output file
```