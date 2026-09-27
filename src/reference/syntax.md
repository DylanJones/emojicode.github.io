# Syntax

>!N **AI-edited:** Parts of this page were written or revised by AI (Claude) to document changes in this fork of
>!N Emojicode, and have not been fully reviewed by a human. If something here disagrees with the compiler, the
>!N compiler is right.

The Language Reference & Guide aims to be – as the title suggests – a reference
and guide in one. Since a programming language needs formal definitions, you’ll
see syntactic definitions from time to time. The notation used and the most
basic syntactic definitions are described in this chapter. The meaning of these
structures will be described in detail in the following chapters.

>!H If you don’t really care about syntactic definitions, no worries! You should
>!H be able to follow along without problems. Just skip this chapter and ignore
>!H them.

Note that the grammar specified in the Language Reference & Guide is not a
complete description of the Emojicode language. The grammar alone allows
programs that are not valid. The accompanying text will outline further rules
that programs must obey.

## The Formal Grammar

The grammar in this Language Reference & Guide is written for people, so it
leaves out details and sometimes simplifies. If you need to know exactly what
the compiler accepts, for instance because you’re writing a tool that reads
Emojicode, have a look at the formal grammar in the compiler repository:
[`docs/grammar.ebnf`](https://github.com/DylanJones/emojicode/blob/master/docs/grammar.ebnf).

The formal grammar is written in the W3C EBNF notation that the XML
specification uses and has two levels:

- The *lexical grammar* describes how a document is divided into tokens:
  whitespace and comments, emoji, identifiers, variables, numbers, strings and
  the other kinds of tokens.
- The *syntactic grammar* describes which sequences of tokens form a valid
  document, from type definitions down to expressions. It includes the
  precedence of the operators.

It describes exactly the documents that the compiler’s lexer and parser accept.
Only errors that depend on more than the sequence of tokens are out of its
scope, for instance a type or method that is declared twice, more than one 🏁
block in a package, or a 📦 import or 📜 include that names a package or file
that cannot be found or read. Rules the compiler checks later, like type
checking, aren’t part of the grammar either.

The grammar is not only documentation: `tools/grammar_check.py` checks it
against the compiler. It compares the grammar’s tables of emoji and keywords
with the compiler’s source code, and compares which documents the grammar and
the compiler accept, using the compiler’s own
[`--dump-tokens` and `--parse-only` options](compiler.html#printing-the-tokens-of-a-file)
on the Emojicode files in the compiler’s repository and its tests, on randomly
changed copies of them and on documents generated from the grammar. If you build the compiler
yourself, `ninja grammar` runs this check.

>!H Where this Language Reference & Guide and `docs/grammar.ebnf` disagree
>!H about syntax, `docs/grammar.ebnf` is right – and where the grammar and the
>!H compiler disagree, the compiler is.

## Notation

The Language Reference & Guide uses a modified BNF grammar notation. Consider
this example:

<pre class="syntax">
<span class="syntax-placeholder">hippo</span> ⟶ <span class="syntax-placeholder">rhinoceros</span> 🥘
<span class="syntax-placeholder">panther</span> ⟶ [🍞] <span class="syntax-placeholder">hyena</span> | 🍮
</pre>

The first line of the above example defines a rule called *hippo*, which
states that a *hippo* consists of a *rhinoceros*, which is another rule as
indicated by the purple background, and the emoji 🥘.

The grammar notation we use in the documentation follows these rules:

- Every rule begins with a name and ⟶ (read “consists of”).
- A vertical bar (`|`) is used to separate alternatives. E.g.

  <pre class="syntax">
  <span class="syntax-placeholder">foo</span> ⟶ <span class="syntax-placeholder">mouse</span> <span class="syntax-placeholder">dog</span> | <span class="syntax-placeholder">hippo</span>
  </pre>

  denotes that a *foo* may consist of a *mouse* and a *dog* or of only a *hippo*.

- Rules can be broken into multiple lines starting with the same name, the new
  line then is an alternative as if the contents right to the ⟶ was on the same
  line with previous definition separated by a vertical bar. E.g.

  <pre class="syntax">
  <span class="syntax-placeholder">foo</span> ⟶ <span class="syntax-placeholder">mouse</span> <span class="syntax-placeholder">dog</span> | <span class="syntax-placeholder">hippo</span>
  </pre>

  and

  <pre class="syntax">
  <span class="syntax-placeholder">foo</span> ⟶ <span class="syntax-placeholder">mouse</span> <span class="syntax-placeholder">dog</span>
  <span class="syntax-placeholder">foo</span> ⟶ <span class="syntax-placeholder">hippo</span>
  </pre>

  indicate the same thing.

- Parts enclosed in square brackets (`[` and `]`) are optional: they can occur
  but do not have to.

- Whitespaces are never terminals but only appear to improve formatting.

- The not sign (`￢`) followed by a terminal or non-terminal indicates that the
  terminal or non-terminal may not occur here even though
  the next non-terminal that is not preceeded by a not sign indicates it could
  occur.

- Terminals beginning with the character sequence `U+` substitue the character
  with an Unicode code point. If two code points are connected with a `–` this
  indicates that all characters in this range shall be allowed.

## Source Text

Emojicode documents are read as UTF-8. Bytes that are not valid UTF-8 are read
as U+FFFD REPLACEMENT CHARACTER (�) instead of stopping the compiler. Since
U+FFFD is not an emoji, it usually ends up in a variable name or in a string.

U+FE0F VARIATION SELECTOR-16 is the invisible character that many keyboards
add after an emoji to ask for its colourful presentation, e.g. ❗️ is ❗
followed by U+FE0F. The compiler treats U+FE0F like whitespace and ignores it
inside emoji identifiers and operators, so it makes no difference whether an
emoji is written with or without it: ❗️ and ❗ are the same token, and so are 🖍️
and 🖍. Where the grammar in this Language Reference & Guide contains an emoji,
it can therefore be written with or without U+FE0F.

## Document Syntax

Every Emojicode source code document consists of any number of
*document-statements*.

```syntax
$document-statement$-> $package-import$ | $include$ | $package-documentation-comment$
$document-statement$-> $type-definition$ | $link-hints$
$document-statement$-> $start-flag$
$include$-> 📜 $string-literal$
$start-flag$-> 🏁 [$return-type$] $block$
```

## Statement and Expression

The smallest standalone elements of Emojicode’s normal program code is called
*statement*.

```syntax
$statement$-> $expression$ | $assignment$ | $declaration$ | $operator-assignment$
$statement$-> $return$ | $error-check-control$ | $raise$ | $method-assignment$
$statement$-> $if$ | $for-in$ | $repeat-while$ | $unsafe-block$
$expression$-> $numeric-literal$ | 👍 | 👎
$expression$-> $collection-literal$ | $string-literal$ | $no-value$
$expression$-> $method-call$ | $unwrap$ | $this$
$expression$-> $operator-expression$ | $group$
$expression$-> $callable-call$ | $closure$
$expression$-> $super$ | $reraise$ | $cast$
$expression$-> $type-value$ | $instantiation$ | $size-of$ | $unique$
```

## Emoji and Variable

```syntax
$unicode$-> <i>any Unicode character that is not whitespace</i>
$variable$-> $variable-head$ [$variable-parts$]
$variable-head$-> --$integer-prefix$ --$number$ --$emoji$ $unicode$
$variable-parts$-> $variable-part$ | $variable-part$ $variable-parts$
$variable-part$-> --$emoji$ $unicode$
```

## Numeric Literals

```syntax
$numeric-literal$-> $integer-literal$ $float-literal$
$integer-literal$-> [$integer-prefix$] $integer-number$
$integer-prefix$-> + | - | 0x
$float-literal$-> [$float-prefix$] $numbers$
$float-prefix$-> + | -
$integer-numbers$-> $integer-number$ | $integer-number$ $integer-numbers$
$integer-number$-> $number$ | a | b | c | d | e | f | A | B | C | D | E | F
$numbers$-> $number$ | $number$ $numbers$
$number$-> 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
```

## Emoji

The compiler recognizes the emoji of Unicode Emoji 18.0. Newer emoji like 🫠,
🥱 or 🪿 can therefore be used in names, and an emoji with a skin tone
modifier, like 🫶🏾, is a single emoji.

Emoji can be joined into one name with U+200D ZERO WIDTH JOINER, as in many
emoji sequences, or with 🔸. The first emoji after a joiner always belongs to
the name, even if it is 🔸 itself: `↘🔸🔸🔡` is the name `↘🔸🔸` followed by
the name `🔡`.

```syntax
$emoji$-> $emoji-main$ | $emoji-modifier-base$ $emoji-modifier$ | $zwj$ | $regional-indicator$ $regional-indicators$
$zwj$-> $emoji$ U+200D $emoji$
$emoji-main$-> $emoji-main$ U+FE0F
$emoji-main$-> U+00A9 | U+00AE | U+203C | U+2049 | U+2122
$emoji-main$-> U+2139 | U+2194–U+2199 | U+21A9–U+21AA | U+231A–U+231B | U+2328
$emoji-main$-> U+23CF | U+23E9–U+23F3 | U+23F8–U+23FA | U+24C2 | U+25AA–U+25AB
$emoji-main$-> U+25B6 | U+25C0 | U+25FB–U+25FE | U+2600–U+2604 | U+260E
$emoji-main$-> U+2611 | U+2614–U+2615 | U+2618 | U+261D | U+2620
$emoji-main$-> U+2622–U+2623 | U+2626 | U+262A | U+262E–U+262F | U+2638–U+263A
$emoji-main$-> U+2640 | U+2642 | U+2648–U+2653 | U+265F–U+2660 | U+2663
$emoji-main$-> U+2665–U+2666 | U+2668 | U+267B | U+267E–U+267F | U+2692–U+2697
$emoji-main$-> U+2699 | U+269B–U+269C | U+26A0–U+26A1 | U+26A7 | U+26AA–U+26AB
$emoji-main$-> U+26B0–U+26B1 | U+26BD–U+26BE | U+26C4–U+26C5 | U+26C8 | U+26CE–U+26CF
$emoji-main$-> U+26D1 | U+26D3–U+26D4 | U+26E9–U+26EA | U+26F0–U+26F5 | U+26F7–U+26FA
$emoji-main$-> U+26FD | U+2702 | U+2705 | U+2708–U+270D | U+270F
$emoji-main$-> U+2712 | U+2714 | U+2716 | U+271D | U+2721
$emoji-main$-> U+2728 | U+2733–U+2734 | U+2744 | U+2747 | U+274C
$emoji-main$-> U+274E | U+2753–U+2755 | U+2757 | U+2763–U+2764 | U+2795–U+2797
$emoji-main$-> U+27A1 | U+27B0 | U+27BF | U+2934–U+2935 | U+2B05–U+2B07
$emoji-main$-> U+2B1B–U+2B1C | U+2B50 | U+2B55 | U+3030 | U+303D
$emoji-main$-> U+3297 | U+3299 | U+1F004 | U+1F0CF | U+1F170–U+1F171
$emoji-main$-> U+1F17E–U+1F17F | U+1F18E | U+1F191–U+1F19A | U+1F1E6–U+1F1FF | U+1F201–U+1F202
$emoji-main$-> U+1F21A | U+1F22F | U+1F232–U+1F23A | U+1F250–U+1F251 | U+1F300–U+1F321
$emoji-main$-> U+1F324–U+1F393 | U+1F396–U+1F397 | U+1F399–U+1F39B | U+1F39E–U+1F3F0 | U+1F3F3–U+1F3F5
$emoji-main$-> U+1F3F7–U+1F4FD | U+1F4FF–U+1F53D | U+1F549–U+1F54E | U+1F550–U+1F567 | U+1F56F–U+1F570
$emoji-main$-> U+1F573–U+1F57A | U+1F587 | U+1F58A–U+1F58D | U+1F590 | U+1F595–U+1F596
$emoji-main$-> U+1F5A4–U+1F5A5 | U+1F5A8 | U+1F5B1–U+1F5B2 | U+1F5BC | U+1F5C2–U+1F5C4
$emoji-main$-> U+1F5D1–U+1F5D3 | U+1F5DC–U+1F5DE | U+1F5E1 | U+1F5E3 | U+1F5E8
$emoji-main$-> U+1F5EF | U+1F5F3 | U+1F5FA–U+1F64F | U+1F680–U+1F6C5 | U+1F6CB–U+1F6D2
$emoji-main$-> U+1F6D5–U+1F6D9 | U+1F6DC–U+1F6E5 | U+1F6E9 | U+1F6EB–U+1F6EC | U+1F6F0
$emoji-main$-> U+1F6F3–U+1F6FC | U+1F7E0–U+1F7EB | U+1F7F0 | U+1F90C–U+1F93A | U+1F93C–U+1F945
$emoji-main$-> U+1F947–U+1F9FF | U+1FA70–U+1FA7C | U+1FA80–U+1FAC6 | U+1FAC8 | U+1FACC–U+1FADD
$emoji-main$-> U+1FADF–U+1FAEB | U+1FAEF–U+1FAFA
$emoji-modifier-base$-> $emoji-modifier-base$ U+FE0F
$emoji-modifier-base$-> U+261D | U+26F9 | U+270A–U+270D | U+1F385 | U+1F3C2–U+1F3C4
$emoji-modifier-base$-> U+1F3C7 | U+1F3CA–U+1F3CC | U+1F442–U+1F443 | U+1F446–U+1F450 | U+1F466–U+1F478
$emoji-modifier-base$-> U+1F47C | U+1F481–U+1F483 | U+1F485–U+1F487 | U+1F48F | U+1F491
$emoji-modifier-base$-> U+1F4AA | U+1F574–U+1F575 | U+1F57A | U+1F590 | U+1F595–U+1F596
$emoji-modifier-base$-> U+1F645–U+1F647 | U+1F64B–U+1F64F | U+1F6A3 | U+1F6B4–U+1F6B6 | U+1F6C0
$emoji-modifier-base$-> U+1F6CC | U+1F90C | U+1F90F | U+1F918–U+1F91F | U+1F926
$emoji-modifier-base$-> U+1F930–U+1F939 | U+1F93C–U+1F93E | U+1F977 | U+1F9B5–U+1F9B6 | U+1F9B8–U+1F9B9
$emoji-modifier-base$-> U+1F9BB | U+1F9CD–U+1F9CF | U+1F9D1–U+1F9DD | U+1FAC3–U+1FAC5 | U+1FAF0–U+1FAFA
$emoji-modifier$-> U+1F3FB-U+1F3FF
$regional-indicators$-> $regional-indicator$ [$regional-indicators$]
$regional-indicator$-> U+1F1E6-U+1F1FF
$emoji-id$-> --$binary-operator$ --👍 --👎 --🔟 --🆕 --❗️ --❓--🤛 --🤜 --↩️ --⤴️ --🔂 --🔁 --🚨 --↪️ --🙅 --🍉 --🍇 --🆗 --👇 --☣️ --➡️ --⬅️ --🖍 --🐇 --🐊 --🦃 --🕊 --📗 --🔤 --🎍 $emoji$ | $multi-emoji$
$multi-emoji$-> $emoji-id$ 🔸 $emoji-id$
```

## Decorators

Decorators are a combination of 🎍 and another emoji and are used for keywords
that have a very specific meaning or are seldom used.

```syntax
$decorator$-> 🎍 $emoji-main$
```
