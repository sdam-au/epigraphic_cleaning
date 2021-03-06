Epigraphic cleaning models
================
Petra Hermankova
05/05/2020

*Setting up the environment*

``` r
knitr::opts_chunk$set(echo = TRUE, error=TRUE)
devtools::install_github("mplex/cedhar", subdir="pkg/sdam")
#install.packages("rjson")
#install.packages("tidyverse")
#install.packages("getPass")
#install.packages("formatR")
install.packages("rmarkdown")

library(tidyverse)
library(tidytext)
library(dplyr)
library(stringr)
library(sdam)
library(rjson)
library(getPass)
library(formatR)
```

# Cleaning epigraphic text for tidy text mining analysis (word and sentence centered)

*Aim:* The main purpose of this script is to clean large collections of
epigraphic texts all at once in order to create cleaned texts ready for
text mining analysis. The output clean texts can be used for a) word
centered text mining, also known as the tidytext approach
(<https://www.tidytextmining.com/>) or for b) sentence centered text
mining, as part of the Natural Language Processing
(<https://en.wikipedia.org/wiki/Natural_language_processing>).

The presented cleaning process is designed as generic, fairly modular
and fully customisable. Therefore, it can be with some modification used
to any epigraphic corpus. Ample examples are provided to illustrate
individual parts of the process, so anyone familiar with *Regular
Expressions* and *basic understanding of R* can build their own cleaning
function or modify the existing ones.

The final output of the cleaning function depends on which of the
individual cleaning blocks will be used and in what sequence they will
run. Each individual cleaning block represents one pattern occurring
repeatedly in the text that can be searched for and modified, depending
on the intended outcome. All the cleaning steps are dependent on the
characteristics of the original dataset, therefore familiarity with the
original dataset prior the cleaning process is recommended. Each dataset
can have a different set of symbols and characters to be cleaned, thus,
the cleaning blocks should be adjusted accordingly.

I have created three categories of cleaning blocks, closely linked with
the methodological approach and the purpose of the cleaning process:

1.  `Conservative cleaning model` producing a text as close to the
    original as possible
2.  `Interpretive cleaning model` producing a text enriched with
    interpretations of the corpus editor
3.  Generic cleaning of patterns common for both previous categories

*Structure of a cleaning block:*

Each of the cleaning blocks maintains the same structure, using Regular
expressions to find and replace the searched term or pattern.

`regexpatternname <- c("regexpattern", "substitutionpattern")`

## 1\. Cleaning blocks for the conservative model

*The aim of this model is to produce a clean text that is as close to
the original text of an inscription as possible, without any editorial
input.*

The cleaned output of the conservative model will be as close to the
original text of an inscription as possible. In most cases it should
resemble a *diplomatic edition* of epigraphic text with spaces between
words, lowercase letters, eliminated brackets and non-utf compliant
symbols. The interpretive restoration, substitutions or any changes of
the text as appear in the dataset, done by the editor of the epigraphic
corpus, are eliminated from the conservative model.

### 1.1. Expanded abbreviations

**Aim:** All expanded abbreaviations that are in the parenthesis () will
be eliminated from the clean text (substituted with "").

  - Example before cleaning: `Αὐρ(ήλιος) Οὐαλέριος`
  - Example after cleaning: `Αὐρ Οὐαλέριος`

<!-- end list -->

``` r
expanded_abbreviations_conservative <- c("\\([^(]*\\)", "")
```

### 1.2. Suppresion of a text with superscripts

**Aim:** All supressions that are in the curly braces {} followed by one
or more superscript digits will be eliminated from the clean text
(substituted with "").

**\!\!\!** It is crutial that block `suppresion_conservative` does not
precede block `suppresion_superscripts_conservative`, otherwise the
Regex pattern would not clean the text properly. This particular pattern
is common for the PHI dataset and may or may not appear in other
datasets.

  - Example before cleaning: `ἱερεὺς ληφθὶς ὑπὰ {²⁶ὑπὸ}²⁶ τῶν βαρβάρων`
  - Example after cleaning: `ἱερεὺς ληφθὶς ὑπὰ τῶν βαρβάρων`

<!-- end list -->

``` r
suppresion_superscripts_conservative <- c("{[^}]*}[⁰¹²³⁴⁵⁶⁷⁸⁹]+", "")
```

### 1.3. Suppresion of a text

**Aim:** All curly braces {} will be eliminated from the clean text
(substituted with ""), while the contents of the braces will remain in
the text.

**\!\!\!** It is crutial that block `suppresion_conservative` does not
precede block `suppresion_superscripts_conservative`, otherwise the
Regex pattern would not clean the text properly.

  - Example before cleaning: `Σεβαστοῦ υἱοῦ {θ̣εοῦ Σεβαστοῦ} τύχης`
  - Example after cleaning: `Σεβαστοῦ υἱοῦ θ̣εοῦ Σεβαστοῦ τύχης`

<!-- end list -->

``` r
suppresion_conservative <- c("[\\{*\\}]", "")
```

### 1.4. Restoration

**Aim:** All restoration that are in the square brackets \[\] will be
eliminated from the clean text (substituted with "").

**\!\!\!** Beware that by eliminating the contents of the brackets you
may loose some context - use at your own discretion.

  - Example before cleaning: `[Ν]ανα Ἕλληνο̣[ς] θυγάτηρ καὶ ἡ ἑτέρα
    [γυνὴ]`
  - Example after cleaning: `ανα Ἕλληνο θυγάτηρ καὶ ἡ ἑτέρα`

<!-- end list -->

``` r
restoration_conservative <- c("\\[[^[]*\\]", "")
```

### 1.5. Substitution

**Aim:** All substitutions that are in the angular brackets \<\> will be
eliminated from the clean text (substituted with "").

**\!\!\!** Beware that by eliminating the contents of the brackets you
may loose some context - use at your own discretion.

  - Example before cleaning: `κωρο<ν Ἀ>ντιόχ<ου> ἡ πατρὶς τειμῆ<ς>`
  - Example after cleaning: `κωρο ντιόχ ἡ πατρὶς τειμῆς`

<!-- end list -->

``` r
substitution_conservative <- c("\\<[^<]*\\>", "")
```

### 1.6. Substitution in EDH dataset

**Aim:** All sustitutions following the pattern “A=B” will be cleaned
thw following way: B remain in the text and the equal sign and A will be
eliminated from the clean text.

**\!\!\!** Beware that by eliminating the brackets you may loose some
information about the preservation of the text - use at your own
discretion. The `substitution_edh_interpretive` should be run before
`substitution_interpretive` block, otherwise the Regex pattern would not
clean the text properly. The `substitution_interpretive` block will
clean the angular brackets in the next step.

  - Example before cleaning: `pos<u=I>erunt bene merenti`
  - Example after cleaning: `pos<I>erunt bene
merenti`

<!-- end list -->

``` r
substitution_edh_conservative <- c("([α-ωΑ-Ωa-zA-Z])=([α-ωΑ-Ωa-zA-Z])", "\\2")
```

## 2\. Cleaning blocks for the interpretive model

*The aim of this model is to produce a clean text that is enriched with
interpretations of the original text as published by the editor of the
corpus. The editorial interpretations include abbreviations,
restorations, substitutions and suppresions of the text.*

The output of the interpretive model will produce an epigraphic text
with as many editorial suggestions, restorations, corrections, and
improvements as possible to provide as much possible contents of the
inscription as possible. The brackets and non-utf compliant symbols will
be eliminated from the `interpretive model`.

### 2.1. Expanded abbreviations

**Aim:** All parenthesis () will be eliminated from the clean text
(substituted with ""), while the contents of the parenthesis will remain
in the text.

  - Example before cleaning: `Αὐρ(ήλιος) Οὐαλέριος`
  - Example after cleaning: `Αὐρήλιος Οὐαλέριος`

<!-- end list -->

``` r
expanded_abbreviations_interpretive <- c("[\\(*\\)]", "")
```

### 2.2. Suppresion of a text with superscripts

**Aim:** Contents found within curly braces {} followed by one or more
superscript digits will substitute the word immediately preceding the
curly braces with the word contained in the curly braces and the braces
will be eliminated, see example. Note: The cleaning block will not work
if there is no text preceeding the curly braces (the pattern will be
skipped).

**\!\!\!** This particular pattern is common for the PHI dataset and may
or may not appear in other datasets. It is recommended to run the
`suppresion_keep_interpretive` or `suppresion_remove_interpretive` block
after `suppresion_superscripts_interpretive` block, otherwise the Regex
pattern would not clean the text properly.

  - Example before cleaning: `ἱερεὺς ληφθὶς ὑπὰ {²⁶ὑπὸ}²⁶ τῶν βαρβάρων`
  - Example after cleaning: `ἱερεὺς ληφθὶς ὑπὸ τῶν
βαρβάρων`

<!-- end list -->

``` r
suppresion_superscripts_interpretive <- c(" [^ ]+ \\{([⁰¹²³⁴⁵⁶⁷⁸⁹]+)([^}]+)\\}\\1", " \\2")
```

### 2.3. Suppresion of a text

**Aim:** All curly braces {} will be eliminated from the clean text
(substituted with ""), while the contents of the braces will remain in
the text.

**\!\!\!** It is crutial that block `suppresion_keep_interpretive` or
`suppresion_remove_interpretive` does not precede block
`suppresion_superscripts_interpretive`, otherwise the Regex pattern
would not clean the text properly. Due to ambiguous use of {} by editors
of epigraphic corpora, the exact usage depends on the specific dataset
and the way the curly braces were used. Therefore, two options how to
handle curly braces are provided: If you wish to keep the text within
the curly braces and remove the braces, use
`suppresion_keep_interpretive` block. If you wish to remove the text in
the braces and the braces, use `suppresion_remove_interpretive` block.

  - Example before cleaning: `θ̣εοῦ Σεβαστοῦ υἱοῦ {θ̣εοῦ Σεβαστοῦ}
    τύχης`
  - Example after cleaning (keep text): `θ̣εοῦ Σεβαστοῦ υἱοῦ θ̣εοῦ
    Σεβαστοῦ τύχης`
  - Example after cleaning (remove text): `θ̣εοῦ Σεβαστοῦ υἱοῦ τύχης`

<!-- end list -->

``` r
suppresion_keep_interpretive <- c("[\\{*\\}]", "")
```

OR if you wish to remove the contents of the braces

``` r
suppresion_remove_interpretive <- c("{[^}]*}", "")
```

### 2.4. Restoration

**Aim:** All square brackets \[\] will be eliminated from the clean text
(substituted with ""), while the contents of the brackets will remain in
the text.

**\!\!\!** Beware that by eliminating the brackets you may loose some
information about the preservation of the text - use at your own
discretion.

  - Example before cleaning: `[Ν]ανα Ἕλληνο̣[ς] θυγάτηρ καὶ ἡ ἑτέρα
    [γυνὴ]`
  - Example after cleaning: `Νανα Ἕλληνο̣ς θυγάτηρ καὶ ἡ ἑτέρα γυνὴ`

<!-- end list -->

``` r
restoration_interpretive <- c("[\\[*\\]]", "")
```

### 2.5. Substitution

**Aim:** All angular brackets \<\> will be eliminated from the clean
text (substituted with ""), while the contents of the brackets will
remain in the text.

**\!\!\!** Beware that by eliminating the brackets you may loose some
information about the preservation of the text - use at your own
discretion.

  - Example before cleaning: `κωρο<ν Ἀ>ντιόχ<ου> ἡ πατρὶς τειμῆ<ς>`
  - Example after cleaning: `κωρον Ἀντιόχου ἡ πατρὶς τειμῆς`

<!-- end list -->

``` r
substitution_interpretive <- c("[\\<*\\>]", "")
```

### 2.6. Substitution in the EDH dataset

**Aim:** All sustitutions following the pattern “A=B” will be cleaned
the following way: “A” will remain in the text and the equal sign and
“B” will be eliminated from the clean text.

**\!\!\!** The `substitution_edh_interpretive` should be run before
`substitution_interpretive` block, otherwise the Regex pattern would not
clean the text properly. The `substitution_interpretive` block will
clean the angular brackets in the next step.

  - Example before cleaning: `pos<u=I>erunt bene merenti`
  - Example after cleaning: `pos<u>erunt bene
merenti`

<!-- end list -->

``` r
substitution_edh_interpretive <- c("([α-ωΑ-Ωa-zA-Z])=([α-ωΑ-Ωa-zA-Z])", "\\1")
```

## 3\. The generic text cleaning

*The aim of the generic cleaning is to strip the epigraphic text any
non-utf compliant symbols and characters that do not adhere to the
principles of a quantitat ive text mining.*

The cleaning blocks in this section represent common patterns appearing
in any epigraphic text, such as interpunction, lacunas or other
representations of an empty space, various editorial notes and comments
in the text itself, that are not relevant to the text mining, erasures,
numerals, and several specific unicode symbols appearing in the original
text. Depending on the characteristics of the originahal dataset and the
intended outcome, anyone can change individial cleaning blocks to better
fit their needs. Through testing is, however, strongly recommended\!

### 3.1. Lacuna 1

**Aim:** All square brackets \[\] containing one or more “—” will be
eliminated from the clean text (substituted with "").

**\!\!\!** The block `lacuna1` should be run before
`restoration_conservative` and `restoration_interpretive` blocks,
otherwise the Regex pattern would not clean the text properly. Note: If
there is a text within the square bracket, e.g. `προύχον[τος — — —]` the
block `lacuna1` will skip the pattern. However, the block
`restoration_interpretive` will eliminate the square brackets, the
script `interpunction_symbols` will clean the “—” and the script
`multi_whitespace` will eliminate the extra whitespaces. Therefore the
blocks should be used in combination and in the indicated sequence:
(1)`restoration_interpretive`, (2)`interpunction_symbols` and
(3)`multi_whitespace`.

  - Example before cleaning: `[— — —]ης θεῷ Φοίβῳ`
  - Example after cleaning: `ης θεῷ Φοίβῳ`

<!-- end list -->

``` r
lacuna1 <- c("\\[[— ]+\\]", "")
```

### 3.2. Lacuna 2

**Aim:** All square brackets \[\] containing one or more “.” will be
eliminated from the clean text (substituted with "").

**\!\!\!** The block `lacuna2` should be run before
`restoration_conservative` and `restoration_interpretive` blocks,
otherwise the Regex pattern would not clean the text properly. Note: If
there is a text within the square bracket, e.g. `προύχον[τος...]` the
block `lacuna2` will skip the pattern. However, the block
`restoration_interpretive` will eliminate the square brackets, the
script `interpunction_symbols` will clean the “.” and the script
`multi_whitespace` will eliminate the extra whitespaces. Therefore the
blocks should be used in combination and in the indicated sequence:
(1)`restoration_interpretive`, (2)`interpunction_symbols` and
(3)`multi_whitespace`.

  - Example before cleaning: `[․․]ω Διὶ καὶ Ἥρᾳ`
  - Example after cleaning: `ω Διὶ καὶ Ἥρᾳ`

<!-- end list -->

``` r
lacuna2 <- c("\\[[․]+\\]", "")
```

### 3.3. Vacat

**Aim:** All instances of the following strings “vacat, vac, vac., v.”
will be replaced by a space (substituted with " "). If there is any
extra whitespace, it will be cleaned by `multi_whitespace` block in the
following steps.

**\!\!\!** If your datasets contains latin inscriptions, you may want to
check whether the `vacat` block is not eliminitating more words than
anticipated, e.g. words containing string “vacat” or “vac”. If so,
adjust the cleaning block accordingly, i.e. remove “vac”, or don’t use
it.

  - Example before cleaning: `Ἡρακλείδα vacat χαῖρε.`
  - Example after cleaning: `Ἡρακλείδα χαῖρε.`

<!-- end list -->

``` r
vacat <- c("(vacat|vac|vac\\.|v\\.)", " ")
```

### 3.4. Editorial notes

**Aim:** All instances of the editorial notes in parenthesis such as
(vel sim.) will be replaced by a space (substituted with " "). If there
is any extra whitespace, it will be cleaned by `multi_whitespace` block
in the following steps.

**\!\!\!** The `editorial_notes` block should run before the
`expanded_abbreviations_conservative` and
`expanded_abbreviations_interpretive` blocks, otherwise the Regex
pattern would not clean the text properly.

  - Example before cleaning: `Ἥρωι (vel sim.) Καλλισθένης`
  - Example after cleaning: \`\`Ἥρωι Καλλισθένης\`\`\`

<!-- end list -->

``` r
editorial_notes <-c("\\(vel sim.\\)", " ")
```

### 3.5. New line

**Aim:** All instances of in-line symbol for new line (|) will be
eliminated (substituted with "").

  - Example before cleaning: `Λάμπρη Τ̣ελεσήνορ|ος γυνή.`
  - Example after cleaning: `Λάμπρη Τ̣ελεσήνορος γυνή`

<!-- end list -->

``` r
new_line <- c("[\\||\\/]", "")
```

### 3.6. Split word over two lines

**Aim:** All instances of words split between two lines with a dash (-)
will be eliminated (substituted with "").

  - Example before cleaning: `ἀρχιερέως καὶ εὐποσιάρ-\nχου μηνὸς`
  - Example after cleaning: `ἀρχιερέως καὶ εὐποσιάρχου μηνὸς`

<!-- end list -->

``` r
split_word_multiline <- c("-\\n", "")
```

### 3.7. Erasure empty

**Aim:** All instances of erased text (〚—〛) will be replaced by a space
(substituted with " "). If there is any extra whitespace, it will be
cleaned by `multi_whitespace` block in the following steps.

  - Example before cleaning: `Ἀρτέμιδι 〚— — —〛 ἐπηκόοις.`
  - Example after cleaning: `Ἀρτέμιδι ἐπηκόοις.`

<!-- end list -->

``` r
erasure_empty <- c("〚[— ]+〛", " ")
```

### 3.8. Erasure with new text

**Aim:** All instances of double brackets for erasures (〚 〛) will be
eliminated (substituted with "") and the contents of the double brackets
will be preserved as part of the clean text.

  - Example before cleaning: `Ἀμύντωρ Νουμηνίου 〚χαῖρε〛. καὶ ἡ γυνὴ
    αὐτοῦ`
  - Example after cleaning: `Ἀμύντωρ Νουμηνίου χαῖρε. καὶ ἡ γυνὴ αὐτοῦ`

<!-- end list -->

``` r
erasure_new_text <- c("[〚〛]", "")
```

### 3.9. Dubious dot subscript

**Aim:** All instances of the dubious reading marked by the subscrit dot
(unicode 0323) will be eliminated (substituted with "").

**\!\!\!** The `dubious_dot_subscript` block should happen as first step
of the cleaning, otherwise the letters might shift and the Regex pattern
would not clean the text properly.

  - Example before cleaning: `Ἀ̣πό̣λ̣λ̣ωνος`
  - Example after cleaning: `Ἀπόλλωνος`

<!-- end list -->

``` r
dubious_dot_subscript <- c("\u{0323}", "")
```

### 3.10. Interpunction symbols

**Aim:** All instances of listed interpunction symbols
(,.\!-—\#%^&\*/~:;) will be replaced by a space (substituted with "
"). If there is any extra whitespace, it will be cleaned by
`multi_whitespace` block in the following steps.

**\!\!\!** If you wish to keep sentence separators, such as dots at the
bottom of the line, use `interpunction_keep_sentences` or elimininate
the sentence separators you want to keep in your text from the cleaning
block `interpunction_keep_sentences`.

  - Example before cleaning: `Φιλήτη # θεᾷ Μαλοφόρῳ` or `κεῖμαι
    πρόμοιρος Ἑρμογένης τυμβευθείς. /ἀγὼν`
  - Example after cleaning: `Φιλήτη θεᾷ Μαλοφόρῳ` or `κεῖμαι πρόμοιρος
    Ἑρμογένης τυμβευθείς ἀγὼν`
  - Example after cleaning (keep sentences): `κεῖμαι πρόμοιρος Ἑρμογένης
    τυμβευθείς.
ἀγὼν`

<!-- end list -->

``` r
interpunction_symbols <- c("[,|\\.|․|:|⋮|⁙|;|!|\\-|—|–|#|%|\\^|&|\\*|~|@]", " ")
```

OR

if you wish to preserve sentence separators, such as dots

``` r
interpunction_keep_sentences <- c("[!|\\-|—|–|#|%|\\^|&|\\*|~|@]", " ")
```

### 3.11. Superscript numbers

**Aim:** All instances of superscripted numbers will be eliminated
(substituted with "").

**\!\!\!** The `superscript_numbers` should not be run before the
`suppresion_superscripts_conservative` or
`suppresion_superscripts_interpretive` block, otherwise the Regex
pattern would not clean the text properly.

  - Example before cleaning: `Αὐρ(ήλιος) Διονύσιος #⁵⁶ βʹ #⁵⁶`
  - Example after cleaning: `Αὐρ(ήλιος) Διονύσιος # βʹ #`

<!-- end list -->

``` r
superscript_numbers <- c("[⁰¹²³⁴⁵⁶⁷⁸⁹]+", "")
```

### 3.12. Epigraphic symbols

**Aim:** All instances of the listed specialised epigraphic symbols,
such as the haedera (❦), will be eliminated (substituted with "").

  - Example before cleaning: `ἀγαθῆι ❦ τύχηι`
  - Example after cleaning: `ἀγαθῆι τύχηι`

<!-- end list -->

``` r
epigraphic_symbols <-c ("[❦|·|∙|𐆖|⏑|⏓|⏕]", "")
```

### 3.13. Uncertainty symbols

**Aim:** All instances of th elisted symbols marking uncertainty (?)
will be eliminated (substituted with "").

  - Example before cleaning: `χαῖρε?`
  - Example after cleaning: `χαῖρε`

<!-- end list -->

``` r
uncertainty_symbols <-c ("[\\?]", "")
```

### 3.14. End of line

**Aim:** All instances of end of line symbol () will be replaced by
space (substituted with " ").

  - Example before cleaning: `καὶ ἄρξαντα\nτοῦ κοινοῦ`
  - Example after cleaning: `καὶ ἄρξαντα τοῦ κοινοῦ`

<!-- end list -->

``` r
end_line <- c("\\n", " ")
```

### 3.15. Extra blank space

**Aim:** All instances of extra blank space (“ ”) will be replaced by
space (substituted with " ").

  - Example before cleaning: `ἀγαθῆι   τύχηι.`
  - Example after cleaning: `ἀγαθῆι τύχηι.`

<!-- end list -->

``` r
extra_blank <- c("[ ]+", " ")
```

### 3.16. Multi-whitespace

**Aim:** All instances of more then one whitespace " " next to each
other will be eliminated (substituted with "").

**\!\!\!** The `multi_whitespace` should run as the second last cleaning
block to ensure all redundant white spaces are cleaned from the text.

  - Example before cleaning: `Ἡρακλείδα χαῖρε.`
  - Example after cleaning: `Ἡρακλείδα χαῖρε.`

<!-- end list -->

``` r
multi_whitespace <- c("\\s+", " ")
```

### 3.17. Trailing and leading whitespace

**Aim:** All instances of whitespace " " at the beginning and end of the
line will be eliminated (substituted with "").

**\!\!\!** The `whitespace_endline` should run as the last cleaning
block to ensure all redundant white spaces are cleaned from the text.

  - Example before cleaning: `χαῖρε`
  - Example after cleaning: `χαῖρε`

<!-- end list -->

``` r
whitespace_endline <- c("(^\\s|\\s$)", "")
```

### 3.18. Editorial comments in Latin alphabet

**Aim:** All instances of editorial comments in Latin alphabet that are
enclosed in curly braces {} with superscript numbers will be eliminated
(substituted with "").

**\!\!\!** If your dataset contains Latin inscriptions, use this block
with caution. Verify first, that running the block does not eliminate
any necessary information or text. This block has been specifically
designed for the interpretive cleaning of the PHI Greek Inscription
dataset and it should run before `suppresion_superscripts_interpretive`
and `suppresion_interpretive` blocks, otherwise the Regex pattern would
not clean the text properly.

  - Example before cleaning: `ἀγαθῆι τύχηι. {²in parte inferiore altera
    manu incisa est:}² ὑπὲρ τῆς τοῦ`
  - Example after cleaning: `ἀγαθῆι τύχηι. ὑπὲρ τῆς
τοῦ`

<!-- end list -->

``` r
editorial_comments_latin <- c("\\{([⁰¹²³⁴⁵⁶⁷⁸⁹]+)([a-zA-Z0-9][^}]+)\\}\\1", "")
```

### 3.19. Arabic numerals

**Aim:** All instances of arabic numerals (0-9) will be eliminated
(substituted with "").

**\!\!\!** If your dataset contains arabic numerals that you would like
to keep, use this block with caution. Verify first, that running the
block does not eliminate any necessary information or text. This block
has been specifically designed for the interpretive cleaning of the PHI
Greek Inscription dataset and it should run before `multi_whitespace`
and `whitespace_endline` blocks, otherwise the Regex pattern would not
clean the text properly.

  - Example before cleaning: `ἡ γυνὴ αὐτοῦ ΦιλΙ̣ 4 5 καὶ`
  - Example after cleaning: `ἡ γυνὴ αὐτοῦ ΦιλΙ καὶ`

<!-- end list -->

``` r
arabic_numerals <- c("[0-9]+", "")
```

### 3.20 Unclosed brackets

**Aim:** All instances of unclosed brackets will be eliminated
(substituted with "").

**\!\!\!** Use the `unclosed_brackets` block immediately before
`multi_whitespace` and `whitespace_endline` blocks, otherwise the Regex
pattern would not clean the text properly.

  - Example before cleaning: `ummio isenna Xv [`
  - Example after cleaning: `ummio isenna Xv`

<!-- end list -->

``` r
unclosed_brackets <- c("[\\[|\\{|\\(|\\)|\\}|\\]]", "")
```

-----

# Building cleaning functions for specific datasets

When we have established the individual buidling blocks, we can put them
together in the right sequence and build a cleaning function in R for
conservative and interpretive models.

## 1\. PHI Greek Inscriptions dataset

Source: <https://epigraphy.packhum.org/>

### Loading data

First, we need to load the provided test dataset `PHI_IGBulg-I.csv`
located in the `test_data` folder and create an object `dirtytext`
contain the text to be cleaned. Use `getwd()` function to make sure you
are in the right working directory, so the `read_csv` code works for
you. If not, adjust the path.

``` r
getwd()
```

    ## [1] "/home/petra/Github/epigraphic_cleaning/scripts/R"

``` r
text <- read_csv("../../test_data/PHI_IGBulg-I.csv")
dirtytext <- as.data.frame(select(text, hdr2, data))
```

### Conservative model

*Aim:* to have a clean text that is as close to the original inscription
as preserved on the medium.

``` r
cleaning_conservative_phi <- function(epigraphic_dataset){
  clean_text <- gsub(pattern=dubious_dot_subscript[1], replacement=dubious_dot_subscript[2], x=epigraphic_dataset, perl=TRUE)
  clean_text <- gsub(pattern=lacuna1[1], replacement=lacuna1[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=lacuna2[1], replacement=lacuna2[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=vacat[1], replacement=vacat[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=editorial_notes[1], replacement=editorial_notes[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=expanded_abbreviations_conservative[1], replacement=expanded_abbreviations_conservative[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=suppresion_superscripts_conservative[1], replacement=suppresion_superscripts_conservative[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=suppresion_conservative[1], replacement=suppresion_conservative[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=restoration_conservative[1], replacement=restoration_conservative[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=substitution_conservative[1], replacement=substitution_conservative[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=new_line[1], replacement=new_line[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=split_word_multiline[1], replacement=split_word_multiline[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=erasure_empty[1], replacement=erasure_empty[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=erasure_new_text[1], replacement=erasure_new_text[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=interpunction_symbols[1], replacement=interpunction_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=superscript_numbers[1], replacement=superscript_numbers[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=epigraphic_symbols[1], replacement=epigraphic_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=uncertainty_symbols[1], replacement=uncertainty_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=uncertainty_symbols[1], replacement=uncertainty_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=end_line[1], replacement=end_line[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=extra_blank[1], replacement=extra_blank[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=arabic_numerals[1], replacement=arabic_numerals[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=multi_whitespace[1], replacement=multi_whitespace[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=whitespace_endline[1], replacement=whitespace_endline[2], x=clean_text, perl=TRUE)
      return(clean_text)
}
```

#### Example of conservative cleaning:

*Original text of an inscription IGBulg I² 15(3) before cleaning:*

\[— — — — — — — — — — — — — — —\]

\[— — —δόντα καὶ διανομ\]ὰ̣ς̣ τ̣ῇ̣ τ̣ε̣ κ̣ρ̣α̣-

\[τί\]σ̣τ̣ῃ βουλῇ καὶ ἀγορανόμοις καὶ

\[ταῖ\]ς ἑπτὰ φυλαῖς καὶ τοῖς ὑμνοῦσι

τοὺς Σεβαστοὺς καὶ ἀγοραίοις, ἰ-

α̣τροῖς, παιδευταῖς καὶ τοῖς παρε-

{\[πα\]ρ̣ε̣}π̣ιδη̣μήσα̣σιν {²⁶παρεπιδημήσασιν}²⁶ τῆ̣ς̣ Π̣ε̣ντ\[α\]-

\[πόλεως βουλευταῖς — — — — —\]

\[— — — — — — — — — — — — —\]

Output of the `cleaning_conservative`
function:

``` r
example_conservative <- as.data.frame(cleaning_conservative_phi(dirtytext$data))
example_conservative[30,]
```

    ## [1] "ὰς τῇ τε κραστῃ βουλῇ καὶ ἀγορανόμοις καὶ ς ἑπτὰ φυλαῖς καὶ τοῖς ὑμνοῦσι τοὺς Σεβαστοὺς καὶ ἀγοραίοις ἰατροῖς παιδευταῖς καὶ τοῖς παρερεπιδημήσασιν τῆς Πεντ"

### Interpretive model for ‘tidytext’ analysis based on the analysis of words

*Aim:* to have a clean text enriched by editorial interpretations and
reconstructions of the text (to have as rich text of an inscription as
possible).

The output of the function will consist of words separated by one space,
so the data is ready for tidytext analysis. No interpunction will be
left in the text.

``` r
cleaning_interpretive_word_phi <- function(epigraphic_dataset){
  clean_text <- gsub(pattern=dubious_dot_subscript[1], replacement=dubious_dot_subscript[2], x=epigraphic_dataset, perl=TRUE)
  clean_text <- gsub(pattern=lacuna1[1], replacement=lacuna1[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=lacuna2[1], replacement=lacuna2[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=vacat[1], replacement=vacat[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=editorial_notes[1], replacement=editorial_notes[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=editorial_comments_latin[1], replacement=editorial_comments_latin[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=expanded_abbreviations_interpretive[1], replacement=expanded_abbreviations_interpretive[2], x=clean_text, perl=TRUE)
 clean_text <- gsub(pattern=suppresion_superscripts_interpretive[1], replacement=suppresion_superscripts_interpretive[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=suppresion_keep_interpretive[1], replacement=suppresion_keep_interpretive[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=restoration_interpretive[1], replacement=restoration_interpretive[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=substitution_interpretive[1], replacement=substitution_interpretive[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=new_line[1], replacement=new_line[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=split_word_multiline[1], replacement=split_word_multiline[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=erasure_empty[1], replacement=erasure_empty[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=erasure_new_text[1], replacement=erasure_new_text[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=interpunction_symbols[1], replacement=interpunction_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=superscript_numbers[1], replacement=superscript_numbers[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=epigraphic_symbols[1], replacement=epigraphic_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=uncertainty_symbols[1], replacement=uncertainty_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=end_line[1], replacement=end_line[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=extra_blank[1], replacement=extra_blank[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=arabic_numerals[1], replacement=arabic_numerals[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=multi_whitespace[1], replacement=multi_whitespace[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=whitespace_endline[1], replacement=whitespace_endline[2], x=clean_text, perl=TRUE)
      return(clean_text)
}
```

#### Example of interpretive cleaning (word):

*Original text of an inscription IGBulg I² 15(3) before cleaning:*

\[— — — — — — — — — — — — — — —\]

\[— — —δόντα καὶ διανομ\]ὰ̣ς̣ τ̣ῇ̣ τ̣ε̣ κ̣ρ̣α̣-

\[τί\]σ̣τ̣ῃ βουλῇ καὶ ἀγορανόμοις καὶ

\[ταῖ\]ς ἑπτὰ φυλαῖς καὶ τοῖς ὑμνοῦσι

τοὺς Σεβαστοὺς καὶ ἀγοραίοις, ἰ-

α̣τροῖς, παιδευταῖς καὶ τοῖς παρε-

{\[πα\]ρ̣ε̣}π̣ιδη̣μήσα̣σιν {²⁶παρεπιδημήσασιν}²⁶ τῆ̣ς̣ Π̣ε̣ντ\[α\]-

\[πόλεως βουλευταῖς — — — — —\]

\[— — — — — — — — — — — — —\]

Output of the `cleaning_interpretive_word`
function:

``` r
example_interpretive_word <- as.data.frame(cleaning_interpretive_word_phi(dirtytext$data))
example_interpretive_word[30,]
```

    ## [1] "δόντα καὶ διανομὰς τῇ τε κρατίστῃ βουλῇ καὶ ἀγορανόμοις καὶ ταῖς ἑπτὰ φυλαῖς καὶ τοῖς ὑμνοῦσι τοὺς Σεβαστοὺς καὶ ἀγοραίοις ἰατροῖς παιδευταῖς καὶ τοῖς παρεπιδημήσασιν τῆς Πενταπόλεως βουλευταῖς"

### Interpretive model for ‘tidytext’ analysis based on the analysis of sentences

*Aim:* to have a clean text enriched by editorial interpretations and
reconstructions of the text (to have as rich text of an inscription as
possible).

The output of the function will consist of words separated by one space,
so the data is ready for tidytext analysis. Sentence separators will be
left in the text, so individual sentences can be analysed separately.
For this reason the block `interpunction_symbols` was substituted by
`interpunction_keep_sentences`.

``` r
cleaning_interpretive_sentence_phi <- function(epigraphic_dataset){
  clean_text <- gsub(pattern=dubious_dot_subscript[1], replacement=dubious_dot_subscript[2], x=epigraphic_dataset, perl=TRUE)
  clean_text <- gsub(pattern=lacuna1[1], replacement=lacuna1[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=lacuna2[1], replacement=lacuna2[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=vacat[1], replacement=vacat[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=editorial_notes[1], replacement=editorial_notes[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=editorial_comments_latin[1], replacement=editorial_comments_latin[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=expanded_abbreviations_interpretive[1], replacement=expanded_abbreviations_interpretive[2], x=clean_text, perl=TRUE)
 clean_text <- gsub(pattern=suppresion_superscripts_interpretive[1], replacement=suppresion_superscripts_interpretive[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=suppresion_keep_interpretive[1], replacement=suppresion_keep_interpretive[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=restoration_interpretive[1], replacement=restoration_interpretive[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=substitution_interpretive[1], replacement=substitution_interpretive[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=new_line[1], replacement=new_line[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=split_word_multiline[1], replacement=split_word_multiline[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=erasure_empty[1], replacement=erasure_empty[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=erasure_new_text[1], replacement=erasure_new_text[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=interpunction_keep_sentences[1], replacement=interpunction_keep_sentences[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=superscript_numbers[1], replacement=superscript_numbers[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=epigraphic_symbols[1], replacement=epigraphic_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=uncertainty_symbols[1], replacement=uncertainty_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=end_line[1], replacement=end_line[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=extra_blank[1], replacement=extra_blank[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=arabic_numerals[1], replacement=arabic_numerals[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=multi_whitespace[1], replacement=multi_whitespace[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=whitespace_endline[1], replacement=whitespace_endline[2], x=clean_text, perl=TRUE)
      return(clean_text)
}
```

#### Example of interpretive cleaning (sentence):

*Original text of an inscription IGBulg I² 24(2) before cleaning:*

`ἀγαθῆι τύχηι.`

`Διὶ Δολιχαίῳ`

`Μ(ᾶρκος) Πομπ[έϊ]ο̣ς Λού-`

`κιος βενε̣[φ]ικιά-`

`ρ̣ιος ὑπατικ̣οῦ`

`λεγ(ιῶνος) ∙ αʹ ∙ Ἰταλικῆ̣ς`

`Ἀντωνεινιανῆς,`

`βουλευτὴς Διονυ-`

`σοπολειτῶν, Καλ-`

`λατιανῶν, Μαρ-`

`κιανοπολειτῶν`

`εὐχαριστήριον.`

`ὑπὲρ σωτηρίας τοῦ κυ-`

`ρίου Αὐτοκρά-`

`τορος.`

Output of the `cleaning_interpretive_sentence`
function:

``` r
example_interpretive_sentence <- as.data.frame(cleaning_interpretive_sentence_phi(dirtytext$data))
example_interpretive_sentence[42,]
```

    ## [1] "ἀγαθῆι τύχηι. Διὶ Δολιχαίῳ Μᾶρκος Πομπέϊος Λούκιος βενεφικιάριος ὑπατικοῦ λεγιῶνος αʹ Ἰταλικῆς Ἀντωνεινιανῆς, βουλευτὴς Διονυσοπολειτῶν, Καλλατιανῶν, Μαρκιανοπολειτῶν εὐχαριστήριον. ὑπὲρ σωτηρίας τοῦ κυρίου Αὐτοκράτορος."

### Saving all three versions of the cleaned text in a CSV

Save the output of `cleaning_conservative` and
`cleaning_interpretive_word` and `cleaning_interpretive_sentence`
function together with the original contents of the dataset.

``` r
clean_text <- text %>%
  mutate(clean_text_conservative = cleaning_conservative_phi(text$data)) %>%
  mutate(clean_text_interpretive_word = cleaning_interpretive_word_phi(text$data)) %>% 
  mutate(clean_text_interpretive_sentence = cleaning_interpretive_sentence_phi(text$data))
```

Create a new directory `outputs` in the root folder if it does not exist
and save it as CSV.

``` r
# dir.create("../../outputs")
write_csv(clean_text, path = "../../outputs/PHI_IGBulg-I_clean_text.csv")
```

## 2\. EDH Inscriptions dataset

Source: <https://edh-www.adw.uni-heidelberg.de/>

### Loading data

First, we need to install several more packages and load the libraries
in order to connect to Sciencedata.dk and access the dataset.

1.  Input your sciencedata.dk username - type directly into the RStudio
    console

<!-- end list -->

    ## your sciencedata username:

2.  Make the request (you will be asked for password in a new pop-up
    window)

<!-- end list -->

    ## Please enter password in TK window (Alt+Tab)

Sample data for testing (5000 inscriptions only)

    ## Please enter password in TK window (Alt+Tab)

3.  Make a list from the request and display the first six records
    (head)

<!-- end list -->

``` r
list_json <- fromJSON(resp)
```

    ## Error in fromJSON(resp): unexpected character '<'

``` r
EDH_tibble = as_tibble(list_json)
```

    ## Error in as_tibble(list_json): object 'list_json' not found

``` r
head(EDH_tibble)
```

    ## Error in head(EDH_tibble): object 'EDH_tibble' not found

### Conservative model

*Aim:* to have a clean text that is as close to the original inscription
as preserved on the medium - in case of the EDH dataset column
`diplomatic_text` should be similar to the output of the
`conservative_cleaning` model.

Since the dataset is mostly in Latin, I did not use the following
cleaning scripts: `vacat`, `editorial_notes`, `editorial_comments_latin`
since they would eliminate some parts of the text that should not be
eliminated. I am not using the `suppresion_superscripts_conservative`
script beacuse the structure of the EDH dataset does not contain curly
braces followed by superscript numbers. The script `unclosed_brackets`
has been added since EDH dataset contains a lot of unclosed brackets of
all kinds. Script `substitution_edh_conservative` was added to clean
additional substitution features of the EDH dataset.

``` r
cleaning_conservative_edh <- function(epigraphic_dataset){
  clean_text <- gsub(pattern=dubious_dot_subscript[1], replacement=dubious_dot_subscript[2], x=epigraphic_dataset, perl=TRUE)
  clean_text <- gsub(pattern=lacuna1[1], replacement=lacuna1[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=lacuna2[1], replacement=lacuna2[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=expanded_abbreviations_conservative[1], replacement=expanded_abbreviations_conservative[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=suppresion_conservative[1], replacement=suppresion_conservative[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=restoration_conservative[1], replacement=restoration_conservative[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=substitution_edh_conservative[1], replacement=substitution_edh_conservative[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=substitution_conservative[1], replacement=substitution_conservative[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=new_line[1], replacement=new_line[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=split_word_multiline[1], replacement=split_word_multiline[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=erasure_empty[1], replacement=erasure_empty[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=erasure_new_text[1], replacement=erasure_new_text[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=interpunction_symbols[1], replacement=interpunction_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=superscript_numbers[1], replacement=superscript_numbers[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=epigraphic_symbols[1], replacement=epigraphic_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=uncertainty_symbols[1], replacement=uncertainty_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=uncertainty_symbols[1], replacement=uncertainty_symbols[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=end_line[1], replacement=end_line[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=extra_blank[1], replacement=extra_blank[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=arabic_numerals[1], replacement=arabic_numerals[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=unclosed_brackets[1], replacement=unclosed_brackets[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=multi_whitespace[1], replacement=multi_whitespace[2], x=clean_text, perl=TRUE)
  clean_text <- gsub(pattern=whitespace_endline[1], replacement=whitespace_endline[2], x=clean_text, perl=TRUE)
  return(clean_text)
}
```

#### Example of conservative cleaning:

`Transcription` column of the first five inscriptions before
    cleaning:

``` r
print(EDH_tibble$transcription[1:5])
```

    ## Error in print(EDH_tibble$transcription[1:5]): object 'EDH_tibble' not found

`Diplomatic_text` column of the first five inscriptions (for comparison
with the cleaning
    output):

``` r
print(EDH_tibble$diplomatic_text[1:5])
```

    ## Error in print(EDH_tibble$diplomatic_text[1:5]): object 'EDH_tibble' not found

Output of the `cleaning_conservative_edh`
function:

``` r
example_edh <- as.data.frame(cleaning_conservative_edh(EDH_tibble$transcription))
```

    ## Error in gsub(pattern = dubious_dot_subscript[1], replacement = dubious_dot_subscript[2], : object 'EDH_tibble' not found

``` r
example_edh[1:5,]
```

    ## Error in eval(expr, envir, enclos): object 'example_edh' not found

### Interpretive model for ‘tidytext’ analysis based on the analysis of words

*Aim:* to have a clean text enriched by editorial interpretations and
reconstructions of the text (to have as rich text of an inscription as
possible).

Since the dataset is mostly in Latin, I did not use the following
cleaning scripts: `vacat`, `editorial_notes`, `editorial_comments_latin`
since they would eliminate some parts of the text that should not be
eliminated. I am not using the `suppresion_superscripts_interpretive`
script beacuse the structure of the EDH dataset does not contain curly
braces followed by superscript numbers. The script `unclosed_brackets`
has been added since EDH dataset contains a lot of unclosed brackets of
all kinds. Script `substitution_edh_interpretive` was added to clean
additional substitution features of the EDH dataset.

EDH has provided their own version of clean text in the column
`text_cleaned` but did not provide any cleaning script or steps leading
to the current state of `text_cleaned`. As a second step I will compare
the output of the `interpretive_cleaning` model with the `text_cleaned`
version to see who has produced better text for text mining.

The output of the function will consist of words separated by one space,
so the data is ready for tidytext analysis. No interpunction will be
left in the text.

``` r
cleaning_interpretive_word_edh <- function(epigraphic_dataset){
   clean_text <- gsub(pattern=dubious_dot_subscript[1], replacement=dubious_dot_subscript[2], x=epigraphic_dataset, perl=TRUE)
   clean_text <- gsub(pattern=lacuna1[1], replacement=lacuna1[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=lacuna2[1], replacement=lacuna2[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=expanded_abbreviations_interpretive[1], replacement=expanded_abbreviations_interpretive[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=suppresion_keep_interpretive[1], replacement=suppresion_keep_interpretive[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=restoration_interpretive[1], replacement=restoration_interpretive[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=substitution_edh_interpretive[1], replacement=substitution_edh_interpretive[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=substitution_interpretive[1], replacement=substitution_interpretive[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=new_line[1], replacement=new_line[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=split_word_multiline[1], replacement=split_word_multiline[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=erasure_empty[1], replacement=erasure_empty[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=erasure_new_text[1], replacement=erasure_new_text[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=interpunction_symbols[1], replacement=interpunction_symbols[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=superscript_numbers[1], replacement=superscript_numbers[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=epigraphic_symbols[1], replacement=epigraphic_symbols[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=uncertainty_symbols[1], replacement=uncertainty_symbols[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=end_line[1], replacement=end_line[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=extra_blank[1], replacement=extra_blank[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=arabic_numerals[1], replacement=arabic_numerals[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=multi_whitespace[1], replacement=multi_whitespace[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=whitespace_endline[1], replacement=whitespace_endline[2], x=clean_text, perl=TRUE)
      return(clean_text)
}
```

`Transcription` column of the first five inscriptions before
    cleaning:

``` r
print(EDH_tibble$transcription[1:5])
```

    ## Error in print(EDH_tibble$transcription[1:5]): object 'EDH_tibble' not found

`Text_cleaned` column, provided by EDH as a clean version of the text,
for comparison with the output of the `cleaning_intepretive_word_edh`
function:

``` r
print(EDH_tibble$text_cleaned[1:5])
```

    ## Error in print(EDH_tibble$text_cleaned[1:5]): object 'EDH_tibble' not found

Output of the `cleaning_interpretive_word_edh`
function:

``` r
example_edh2 <- as.data.frame(cleaning_interpretive_word_edh(EDH_tibble$transcription))
```

    ## Error in gsub(pattern = dubious_dot_subscript[1], replacement = dubious_dot_subscript[2], : object 'EDH_tibble' not found

``` r
example_edh2[1:5,]
```

    ## Error in eval(expr, envir, enclos): object 'example_edh2' not found

### Interpretive model for ‘tidytext’ analysis based on the analysis of sentences

*Aim:* to have a clean text enriched by editorial interpretations and
reconstructions of the text (to have as rich text of an inscription as
possible).

Since the dataset is mostly in Latin, I did not use the following
cleaning scripts: `vacat`, `editorial_notes`, `editorial_comments_latin`
since they would eliminate some parts of the text that should not be
eliminated. I am not using the `suppresion_superscripts_interpretive`
script beacuse the structure of the EDH dataset does not contain curly
braces followed by superscript numbers. The script `unclosed_brackets`
has been added since EDH dataset contains a lot of unclosed brackets of
all kinds. Script `substitution_edh_interpretive` was added to clean
additional substitution features of the EDH dataset.

EDH has provided their own version of clean text in the column
`text_cleaned` but did not provide any cleaning script or steps leading
to the current state of `text_cleaned`. As a second step I will compare
the output of the `interpretive_cleaning` model with the `text_cleaned`
version to see who has produced better text for text mining.

The output of the function will consist of words separated by one space,
so the data is ready for tidytext analysis. Sentence separators will be
left in the text, so individual sentences can be analysed separately.
For this reason the block `interpunction_symbols` was substituted by
`interpunction_keep_sentences`.

``` r
cleaning_interpretive_sentence_edh <- function(epigraphic_dataset){
   clean_text <- gsub(pattern=dubious_dot_subscript[1], replacement=dubious_dot_subscript[2], x=epigraphic_dataset, perl=TRUE)
   clean_text <- gsub(pattern=lacuna1[1], replacement=lacuna1[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=lacuna2[1], replacement=lacuna2[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=expanded_abbreviations_interpretive[1], replacement=expanded_abbreviations_interpretive[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=suppresion_keep_interpretive[1], replacement=suppresion_keep_interpretive[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=restoration_interpretive[1], replacement=restoration_interpretive[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=substitution_edh_interpretive[1], replacement=substitution_edh_interpretive[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=substitution_interpretive[1], replacement=substitution_interpretive[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=new_line[1], replacement=new_line[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=split_word_multiline[1], replacement=split_word_multiline[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=erasure_empty[1], replacement=erasure_empty[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=erasure_new_text[1], replacement=erasure_new_text[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=interpunction_keep_sentences[1], replacement=interpunction_keep_sentences[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=superscript_numbers[1], replacement=superscript_numbers[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=epigraphic_symbols[1], replacement=epigraphic_symbols[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=uncertainty_symbols[1], replacement=uncertainty_symbols[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=end_line[1], replacement=end_line[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=extra_blank[1], replacement=extra_blank[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=arabic_numerals[1], replacement=arabic_numerals[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=multi_whitespace[1], replacement=multi_whitespace[2], x=clean_text, perl=TRUE)
   clean_text <- gsub(pattern=whitespace_endline[1], replacement=whitespace_endline[2], x=clean_text, perl=TRUE)
      return(clean_text)
}
```

`Transcription` column of three inscriptions before
    cleaning:

``` r
print(EDH_tibble$transcription[c(2297, 2444, 3026)])
```

    ## Error in print(EDH_tibble$transcription[c(2297, 2444, 3026)]): object 'EDH_tibble' not found

`Text_cleaned` column, provided by EDH as a clean version of the
    text:

``` r
print(EDH_tibble$text_cleaned[c(2297, 2444, 3026)])
```

    ## Error in print(EDH_tibble$text_cleaned[c(2297, 2444, 3026)]): object 'EDH_tibble' not found

Output of the `cleaning_interpretive_sentence_edh` function (with
interpunction):

``` r
example_edh3 <- as.data.frame(cleaning_interpretive_sentence_edh(EDH_tibble$transcription))
```

    ## Error in gsub(pattern = dubious_dot_subscript[1], replacement = dubious_dot_subscript[2], : object 'EDH_tibble' not found

``` r
example_edh3[c(2297, 2444, 3026),]
```

    ## Error in eval(expr, envir, enclos): object 'example_edh3' not found

### Enriching the full dataset with conservative and interpretive cleaned versions of the text:

``` r
EDH_clean <- EDH_tibble %>%
  mutate(clean_text_conservative = cleaning_conservative_edh(EDH_tibble$transcription)) %>%
  mutate(clean_text_interpretive_word = cleaning_interpretive_word_edh(EDH_tibble$transcription))  %>% 
  mutate(clean_text_interpretive_sentence = cleaning_interpretive_sentence_edh(EDH_tibble$transcription)) 
```

    ## Error in eval(lhs, parent, parent): object 'EDH_tibble' not found

-----

### Selecting a smaller segment for testing

``` r
Thracia <- EDH_clean%>%
  filter(province_label == "Thracia"| province_label == "Thracia?")
```

    ## Error in eval(lhs, parent, parent): object 'EDH_clean' not found

#### Comparing new cleaned text, original transcription and the EDH cleaned text for quality of cleaning

``` r
number <- 3
print(Thracia$clean_text_interpretive_word[number])     # output of cleaning_interpretive_word function
```

    ## Error in print(Thracia$clean_text_interpretive_word[number]): object 'Thracia' not found

``` r
print(Thracia$clean_text_interpretive_sentence[number]) # output of cleaning_interpretive_sentence function
```

    ## Error in print(Thracia$clean_text_interpretive_sentence[number]): object 'Thracia' not found

``` r
print(Thracia$transcription[number])                    # original text to be cleaned  
```

    ## Error in print(Thracia$transcription[number]): object 'Thracia' not found

``` r
print(Thracia$text_cleaned[number])                     # text_cleaned provided by EDH
```

    ## Error in print(Thracia$text_cleaned[number]): object 'Thracia' not found

## Saving as JSON to local space

``` r
# Thracia_json <- rjson::toJSON(Thracia)
#write(Thracia_json, "../../outputs/EDH_Thracia.json")
```

## Writing as JSON to Sciencedata.dk

``` r
EDH_clean_text_json <- rjson::toJSON(EDH_clean)
```

    ## Error in rjson::toJSON(EDH_clean): object 'EDH_clean' not found

``` r
write(EDH_clean_text_json, file="EDH_clean_text_sample.json")
```

    ## Error in cat(x, file = file, sep = c(rep.int(sep, ncolumns - 1), "\n"), : object 'EDH_clean_text_json' not found

``` r
user <- readline("your sciencedata username: ")
```

    ## your sciencedata username:

``` r
request("EDH_clean_text_sample.json", path="/sharingout/648597@au.dk/SDAM_root/SDAM_data/EDH/", 
        method="PUT", cred=c(user, getPass("your sciencedata password: "))) 
```

    ## Please enter password in TK window (Alt+Tab)

    ## Response [https://sciencedata.dk/sharingout/648597@au.dk/SDAM_root/SDAM_data/EDH//EDH_clean_text_sample.json]
    ##   Date: 2020-05-18 08:46
    ##   Status: 401
    ##   Content-Type: text/html; charset=UTF-8
    ## <EMPTY BODY>

``` r
file.remove("EDH_clean_text_sample.json")
```

    ## [1] TRUE
