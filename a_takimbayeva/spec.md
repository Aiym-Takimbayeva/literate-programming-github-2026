Top words

A news site shows a "Trending words" box beside each article. top_words finds the 10 most common words in an article's text, with their counts. Articles may be in English, Russian, or Kazakh.

Interface
Python 3, standard library only. File top_words.py.
top_words(text: str) -> list[tuple[str, int]]
Each tuple is (word, count). The list is sorted as in "Ordering" and has at most 10 items.
If text is not a str (including None), raise TypeError.
Rules
Normalization. Lowercase the text with str.lower(), then apply Unicode NFC normalization. Then replace U+2019 (’) with ASCII '.
Letter. A letter is any character for which str.isalpha() is true. Digits and _ are not letters.
Word. A word is a maximal run of letters. One ASCII apostrophe ' or one ASCII hyphen-minus - is allowed between two letters and is part of the word. An apostrophe or hyphen at the start or end of a run, or next to whitespace or another symbol, is not part of the word.
Everything else separates words. Whitespace, digits, _, all other punctuation (including en/em dashes – —), and symbols. A digit inside a run splits it: abc123def is abc and def. Numbers alone are never words.
No folding. ё and е are different letters. Latin a (U+0061) and Cyrillic а (U+0430) are different letters. No stemming: книга and книги are different words.
No stop words. the, и, және and the like are counted like any word.
Counting. Count exact occurrences of each normalized word in the text.
Ordering. Sort by count, highest first. Break ties by the word in ascending Unicode code point order (Python's default str comparison). Latin sorts before Cyrillic, and я (U+044F) sorts before ә (U+04D9). A word sorts before a longer word it is a prefix of.
Limit. Return the first 10 items of the sorted list, or all of them if there are fewer than 10. The cut happens after sorting, so ties at the 10th place are resolved by rule 8.
No words. If there are no words (empty string, only spaces, digits, or punctuation), return [].
No special handling of URLs, emails or numbers. example.com is example and com.
Examples

Inputs and expected outputs are Python literals. Each row tests the decision named in its second column.

#	Decision	Input	Expected output
1	Baseline: counts, order, tuple list	'cat dog cat bird dog cat'	[('cat', 3), ('dog', 2), ('bird', 1)]
2	Lowercase (no case-sensitive counting)	'The the THE cat'	[('the', 3), ('cat', 1)]
3	Russian	'Привет мир привет'	[('привет', 2), ('мир', 1)]
4	Kazakh letters	'Сәлем әлем сәлем!'	[('сәлем', 2), ('әлем', 1)]
5	Kazakh uppercase letters Қ, Ғ	'ҚАЗАҚСТАН Қазақстан қазақстан'	[('қазақстан', 3)]
6	Punctuation separates words	'hello,world;hello.'	[('hello', 2), ('world', 1)]
7	Digits separate words, numbers are not words	'abc123def 42 abc'	[('abc', 2), ('def', 1)]
8	Underscore separates words	'snake_case snake'	[('snake', 2), ('case', 1)]
9	Inner apostrophe is part of the word	"don't Don't dont"	[("don't", 2), ('dont', 1)]
10	Curly apostrophe equals ASCII one	"don\u2019t don't"	[("don't", 2)]
11	Edge apostrophes (quotes, plural possessive) are dropped	"'hello' hello dogs' dogs"	[('dogs', 2), ('hello', 2)]
12	Inner hyphen is part of the word	'well-known well known'	[('known', 1), ('well', 1), ('well-known', 1)]
13	Russian hyphenated word	'кто-то кто-то'	[('кто-то', 2)]
14	Dashes with spaces and em/en dashes separate	'Москва — столица, Москва – город'	[('москва', 2), ('город', 1), ('столица', 1)]
15	Hyphen at edge or before a digit is dropped	'-cat- cat covid-19 covid'	[('cat', 2), ('covid', 2)]
16	ё is not folded into е	'ёлка елка'	[('елка', 1), ('ёлка', 1)]
17	Latin a and Cyrillic а differ	'a \u0430'	[('a', 1), ('\u0430', 1)]
18	NFC: и + combining breve equals й	'\u0438\u0306 \u0439'	[('\u0439', 2)]
19	No stemming	'книга книги книга'	[('книга', 2), ('книги', 1)]
20	No stop-word filtering	'и и и в в на дом'	[('и', 3), ('в', 2), ('дом', 1), ('на', 1)]
21	Tie order is by code point, not Kazakh alphabet (я before ә)	'ә я'	[('я', 1), ('ә', 1)]
22	Latin before Cyrillic in ties	'я a'	[('a', 1), ('я', 1)]
23	Limit of 10	'a b c d e f g h i j k'	[('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1), ('i', 1), ('j', 1)]
24	Sort first, then cut: tie at 10th place	'k k a b c d e f g h i j j'	[('j', 2), ('k', 2), ('a', 1), ('b', 1), ('c', 1), ('d', 1), ('e', 1), ('f', 1), ('g', 1), ('h', 1)]
25	Newlines and tabs are whitespace	'cat\ncat\tdog'	[('cat', 2), ('dog', 1)]
26	Mixed languages in one text	'cat кот мен cat'	[('cat', 2), ('кот', 1), ('мен', 1)]
27	URL is not special	'example.com'	[('com', 1), ('example', 1)]
28	Empty string	''	[]
29	Only spaces	'   '	[]
30	Only digits, punctuation, hyphens	'123 !!! ---'	[]

Non-str input (rule in Interface): top_words(None) and top_words(b'cat') raise TypeError.