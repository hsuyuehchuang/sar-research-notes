## GitHub Markdown Math Notes

This note records a few practical rules for writing math in Markdown and rendering it correctly on GitHub.

### Delimiter Usage

Use `\biggl` / `\biggr` with `()` or `[]` only. Using `{}` is illegal on GitHub.

### `align` vs `aligned`

- `align` show the number of equations, while `aligned` does not.
- In GitHub Markdown rendering, both not show the number of equation.

$$
\begin{align}
s_\mathrm{test}(\tau,\eta) 
& = A_1 \exp \biggl( - j \frac{ 4 \pi f_0 R(\eta)}{c} \biggr)
\\
& = A_1 \exp \biggl[ - j \frac{ 4 \pi f_0 R(\eta)}{c} \biggr]
\\ 
\end{align}
$$

$$
\begin{aligned}
s_\mathrm{test}(\tau,\eta) 
& = A_1 \exp \biggl( - j \frac{ 4 \pi f_0 R(\eta)}{c} \biggr)
\\
& = A_1 \exp \biggl[ - j \frac{ 4 \pi f_0 R(\eta)}{c} \biggr]
\\ 
\end{aligned}
$$

### Inline Math Spacing

Do not add spaces inside inline math delimiters. 
Should add spaces outside inline math delimiters.
Write inline expressions as `$a+b$`, not `$ a+b $`.

Examples:
- `,$a+b$,`,$a+b$,
    - illegal
- `, $a+b$ ,`, $a+b$ ,
    - legal
- `, $ a+b $ ,`, $ a+b $ ,
    - illegal
