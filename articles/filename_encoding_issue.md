Filenames with composed and decomposed characters
=================================================

When moving files between different system like macOS, Windows and Linux, 
special characters in the file name can give unexpected results.
Consider this example:

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

2 files that visually have the same name but diff claims that the are
named differently.
To understand this lets look closer on what bytes the names actually
consists of.

	$ ls dir_a/file_ö | hexdump -C
	00000000  64 69 72 5f 61 2f 66 69  6c 65 5f c3 b6 0a        |dir_a/file_...|
                                                -----
	$ ls dir_b/file_ö | hexdump -C
	00000000  64 69 72 5f 62 2f 66 69  6c 65 5f 6f cc 88 0a     |dir_b/file_o...|
                                                --------

The bytes encoding the last character are underlined. In the first case
[0xC3, 0xB6] is the last character and in the second case it is
[0x6F, 0xCC, 0x88].
The first case composed (NFC) character is used and the second is
decomposed (NFD) character is used.
This is two ways to represent the same character in
Unicode [https://en.wikipedia.org/wiki/Unicode_equivalence]

To change between NFC and NFD 'convmv' can be used. 'convmv' can convert
filenames between many different encodings but in this case only type of
composed need to be changed. If 'convmv' is run without the argument '--notest'
it will not change anything.

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

In the first example '--nfd' is used meaning that change to decomposed
characters. In the second example '--nfc' is used instead. Meaning change to
composed characters.
