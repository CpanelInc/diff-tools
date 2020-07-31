# Format files

Each file in this directory attempted to tar up the same contents, using
different well-known formats.  Some of the formats could not handle the long
filenames or symlinks that were in the directory, and thus omitted those
entries.

**Note:** Each file is a _tarbomb_; that is, it contains multiple entries at
the root level - they are not nested inside a folder for namespacing upon
extraction.
