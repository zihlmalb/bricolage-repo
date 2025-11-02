# Use Latex for proper equation rendering

There is a good package called asciidoctor-mathematical that allows you to embed mathematical expressions in the form of LaTeX code into your documents. Unfortunately, this tool uses a library called mathematical for rendering. For a long time, no one has worked on this thing anymore, and unfortunately, it is not capable of rendering all expressions cleanly (for examle underbrace). Also the equation numbering has some issues.

I thought to myself: why not use the original, namely LaTeX?

This extension of the original package causes LaTeX to be used for rendering the equations. Currently, only SVG is supported as the target format. It also fixes the numbering issues I had with it. The extensions are available as a patch ([asciidoctor-mathematical-pdflatex.patch](https://github.com/user-attachments/files/22795329/asciidoctor-mathematical-pdflatex.patch)) patching commit 2b877fc589c71fb3b3e4911c5cec65d5b8191e09 of *asciidoctor-mathematical*.

## Installation

Bevore you start you have to install _textlive-full_ and also _pdf2svg_. Consider also the system prequirements described in https://github.com/asciidoctor-contrib/asciidoctor-mathematical.

Do the following:
~~~
git clone https://github.com/asciidoctor-contrib/asciidoctor-mathematical.git
cd asciidoctor-mathematical
git checkout 2b877fc589c71fb3b3e4911c5cec65d5b8191e09
wget https://github.com/user-attachments/files/22795329/asciidoctor-mathematical-pdflatex.patch
git apply asciidoctor-mathematical-pdflatex.patch
~~~

Afterwards build the gem an install.
~~~
gem build asciidoctor-mathematical.gemspec
gem install./asciidoctor-mathematical-0.3.6.gem
~~~

The enhancement supports following parameters:

-  **latex-font-size**: Default is 12. 10, 11, 12 can be set.
-  **mathematical-pdflatex**: to enable latex-rendering

Compile your document using following call:
~~~
asciidoctor-pdf --trace\
     -r asciidoctor-mathematical\
     -a mathematical-format=svg\
     -a mathematical-pdflatex\
     -a latex-font-size=12\
     sample.adoc
~~~

## Results
See the difference between mathematical and pdflatex rendering:

[sample_mathematical.pdf](https://github.com/user-attachments/files/22796323/sample_mathematical.pdf) vs. [sample_latex.pdf](https://github.com/user-attachments/files/22796327/sample_latex.pdf)

or

[underbrace_mathematical.pdf](https://github.com/user-attachments/files/22796334/underbrace_mathematical.pdf) vs. [underbrace_latex.pdf](https://github.com/user-attachments/files/22796335/underbrace_latex.pdf)

## Issues

- Supports only svg
- Only tested with asciidoctor-pdf
- Slow (compile without `-mathematical-pdflatex` for preview, use with `-mathematical-pdflatex` for final rendering).

