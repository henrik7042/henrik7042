Filenames with composed and decomposed characters
=================================================

When moving files between different systems such as macOS, Windows, and Linux, 
special characters in filenames can lead to unexpected results.
Consider the following example:

    $ tree
	.
	├── dir_a
	│   └── file_ö
	└── dir_b
		└── file_ö

	3 directories, 2 files

    $ diff -r dir_a dir_b
	Only in dir_b: file_ö
	Only in dir_a: file_ö

These are two files that visually appear to have the same name, but diff reports
that they are named differently.
To understand why, let’s take a closer look at the bytes that make up
the filenames.

	$ ls dir_a/file_ö | hexdump -C
	00000000  64 69 72 5f 61 2f 66 69  6c 65 5f c3 b6 0a        |dir_a/file_...|
                                                -----
	$ ls dir_b/file_ö | hexdump -C
	00000000  64 69 72 5f 62 2f 66 69  6c 65 5f 6f cc 88 0a     |dir_b/file_o...|
                                                --------

The bytes encoding the last character are underlined. In the first case,
[0xC3, 0xB6] represents the final character, while in the second case it is
represented by [0x6F, 0xCC, 0x88].

In the first case, a composed (NFC) character is used; in the second case,
a decomposed (NFD) character is used. These are two different ways to represent
the same character in Unicode. See more https://en.wikipedia.org/wiki/Unicode_equivalence

To convert between NFC and NFD, the convmv tool can be used. convmv is capable
of converting filenames between many different encodings, but in this case only
the composition form needs to be changed. If convmv is run without the --notest
argument, it will perform a dry run and make no changes.

	$ convmv -f UTF-8 -t UTF-8 -r --nfd .
	Starting a dry run without changes...
	mv "./dir_a/file_ö"	"./dir_a/file_ö"
	No changes to your files done. Would have converted 1 files in 0 seconds.
	Use --notest to finally rename the files.

    $ convmv -f UTF-8 -t UTF-8 -r --nfc .
	Starting a dry run without changes...
	mv "./dir_b/file_ö"	"./dir_b/file_ö"
	No changes to your files done. Would have converted 1 files in 0 seconds.
	Use --notest to finally rename the files.

In the first example, --nfd is used, which converts filenames to decomposed
characters. In the second example, --nfc is used, which converts filenames to
composed characters.
