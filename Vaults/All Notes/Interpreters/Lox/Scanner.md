What does the scanner have:
```rust
pub struct Scanner {
	// file
    source: Vec<u8>,
	// tokens it gethers as it scans the source.
    tokens: Vec<Token>,
	// start of the current token it is scanning.
    start: usize,
	// where we are at when scanning.
    current: usize,
	// Which column we are at in the current line.
    column: usize,
	// Line of the file.
    line: usize,
}
```

### Scan Tokens
The algorithm is simple:
- While not at the end of the file:
	- `set start = current` and call `scan_token`.
- Push `EOF` token to indicate the end of the file was reached.
- Return a copy of  the tokens.

#### Scan Tokens
Store the current column in a local variable, and call `self.advance`.
Now we match the character returned form `self.advance`, if it matches a keyword we just add that token and return back. Depending on if the character matches more than one keywords or identifiers we match it with the next characters and advance as necessary.

#### Advance
Get the character at the current cursor position, advance current and advance column.
Return the character.

#### Add Token
Adds a token with the literal `None`.

#### Add token with literal
Adds a token with the given literal string to the tokens of the struct. Mainly used when reading a *number or string*.
