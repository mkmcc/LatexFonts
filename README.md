# Installing MinionPro and MyriadPro using `fontpro`

[source](https://github.com/sebschub/FontPro)

steps to install:

```
./scripts/makeall MinionPro
./scripts/install ${HOME}/Library/texmf
updmap-user --enable Map=MinionPro.map

./scripts/clean
./scripts/makeall MyriadPro
./scripts/install ${HOME}/Library/texmf
updmap-user --enable Map=MyriadPro.map
```


# Installing Other OTF Fonts to LaTeX using `autoinst`

## Need to use an old autoinst (2018)

get an [old
version](https://github.com/TeX-Live/texlive-source/tree/tags/texlive-2018.2/texk/texlive/linked_scripts/fontools)
from the repo.

Version `2018.2` worked for me.  `2023` and later did not.  I have not
bisected to find where the bugs appeared, but I suspect it was in ~2019


## For fonts with inferiors (subscript) and superiors (superscript)

NB. several features could work as inferiors:
* `subs` (subscript),
* `sinf` (scientific inferiors), and
* `dnom` (denominator).
I picked `subs`.

Run with --dryrun and inspect the log to see what features are
available.

```
./autoinst.pl --encoding=LY1
              --noupdmap
              --verbose
              --dryrun
              fonts/ArnoPro-*.otf
```


for instance, in `autoinst.log`, you might see:
```
./fonts/ArnoPro-Bold.otf
        Name:       ArnoPro-Bold
        Family:     ArnoPro
        Subfamily:
        Width:      regular
        Weight:     bold
        Shape:      roman
        Size:       11-14
        Features:   ... dnom, frac, ..., subs, sups, ...
```

Add the options `--inferiors=subs --fractions` to use these features.


## Fonts Can Clash:

Run with `--dry-run` and all the options you want (`--inferiors`,
`--fractions`, etc.)and look for warnings along the lines of:

```
./autoinst.pl --encoding=LY1
              --noupdmap
              --inferiors=[subs | sinf | dnom]
              --fractions
              --verbose
              --dryrun
              fonts/ArnoPro-*.otf
```

```
[WARNING] I've parsed both ./fonts/ArnoPro-BoldItalicSmText.otf
                     and ./fonts/ArnoPro-ItalicSmText.otf as

./fonts/ArnoPro-BoldCaption.otf
        Name:       ArnoPro-BoldCaption
        Subfamily:  Caption
        Width:      regular
        Weight:     bold
        Shape:      roman
        Size:       5.9-8.5

./fonts/ArnoPro-SmbdCaption.otf
        Name:       ArnoPro-SmbdCaption
        Subfamily:  Caption
        Width:      regular
        Weight:     semibold
        Shape:      roman
        Size:       5.9-8.5

[WARNING] I failed to parse all fonts in a unique way, so I will split
          your font family into multiple subfamilies and try again:

              ArnoPro, ArnoProBoldSm, ArnoProCaption, ArnoProDisplay, ArnoProSm, ArnoProSmText, ArnoProSubhead

          Please check the output!
```

For each clash, you will need to remove one of the offending fonts.  I
removed `ArnoPro-BoldCaption.otf`.


## To Install the Font:

Once you have selected the features and have fixed clashes, you can
install the font.  I chose to install the my home texmf tree:

```
# make and install the files
./autoinst.pl --target=${HOME}/Library/texmf
              --encoding=LY1
              --noupdmap
              --inferiors=[subs | sinf]
              --fractions
              --verbose
              fonts/ArnoPro-*.otf

# enable the map
updmap-user --force --enable Map=ArnoPro.map

# update the tex hash
mktexlsr ~/Library/texmf
```

## To list the ornaments:

1. Identify the ornament font: `find ~/Library/texmf -name
   '*Arno*orn*'` and copy eg `ArnoPro-Regular-orn-u`.
2. run `latex nfssfont`
3. enter `\currfontname=ACaslonPro-Regular-orn-u` when prompted.
4. enter `\action=\table\bye` when prompted.
5. run `dvipdf nfssfont.dvi` to convert to pdf.
6. `mv nfssfont.pdf ArnoPro-ornaments.pdf`


## add the following to sty

```
%% <MM added>
\RequirePackage{etoolbox}


\newcommand{\superscript}[1]{\textsu{#1}}    % Superior
\newcommand{\swash}[1]{{\swshape #1}}        % Swash
\newcommand{\titling}[1]{{\tlshape #1}}      % Titling


\newcommand{\ArnoPro@updatefamily}{%
  \edef\ArnoPro@familyname{ArnoPro-\ArnoPro@figurealign\ArnoPro@figurestyle}%
  \renewcommand*{\rmdefault}{\ArnoPro@familyname}%
  \normalfont
}
\newcommand*{\figurealign}[1]{%
  \edef\ArnoPro@figurealign{#1}%
  \ArnoPro@updatefamily
}
\newcommand{\myfigureversion}[1]{%
  \ifstrequal{#1}{lf}{\edef\ArnoPro@figurestyle{LF}}{%
    \ifstrequal{#1}{lining}{\edef\ArnoPro@figurestyle{LF}}{%
      \ifstrequal{#1}{osf}{\edef\ArnoPro@figurestyle{OsF}}{%
        \ifstrequal{#1}{oldstyle}{\edef\ArnoPro@figurestyle{OsF}}{%
          \PackageError{ArnoProPlus}{Unknown figure version '#1'}{}%
        }%
      }%
    }%
  }%
  \ArnoPro@updatefamily
}

\newcommand{\switchtotabularfigures}{\figurealign{T}}
\newcommand{\switchtoproportionalfigures}{\figurealign{}}
\newcommand{\switchtoliningfigures}{\myfigureversion{lf}}
\newcommand{\switchtooldstylefigures}{\myfigureversion{oaf}}
%% </MM added>

```
