# Half-open slicing and the midpoint

**TO DO: fold this into `sorting_notes.ipynb`, and add the one comment line to the `merge_sort` in
`sorting_algos.ipynb`. This file is the reminder, the notebook is where it should end up.**

## The short version

When you split a list in merge sort, `mid` is a slice bound, not an index.

```python
mid = len(values) // 2
left = values[:mid]
right = values[mid:]
```

Never `(len(values) - 1) // 2`. That one is the middle *index*, and it breaks at length 2.

## Why the `-1` version breaks

For a list of length `n`, the indices run from `0` to `n - 1`, so the middle index really is
`(n - 1) // 2`. That part is not wrong. The mistake is using it as a slice bound.

At `n = 2`, `(n - 1) // 2` is `0`, so:

- `values[:0]` is empty
- `values[0:2]` is the whole list

The recursion gets the same list back, so it never shrinks, and it runs until the stack overflows.

`n // 2` gives `1` at `n = 2`, so the two halves are one element each. Every call gets a strictly
smaller list, and the recursion terminates.

## Index versus count

| Expression | Answers | Value at n = 4 |
| --- | --- | --- |
| `(n - 1) // 2` | which index is in the middle | 1 |
| `n // 2` | how many elements go in the left half | 2 |

A slice bound is always one past the last element you want, so converting an index into a bound is
always `index + 1`. That is exactly where `n // 2` comes from:

```
((n - 1) + 1) // 2  =  n // 2
```

The two formulas agree when `n` is odd and differ by one when `n` is even. That is why the mistake
hides on lists of length 5 or 7 and only shows up on even lengths, with length 2 being fatal.

## Half-open slicing, explained properly

The usual phrasing is "inclusive start, exclusive end", which makes it sound like a patch bolted onto
0-indexing. It is not. Index base and range closedness are two separate choices, and half-open ranges
behave the same way in a 1-indexed language.

What the half-open convention actually buys:

- `len(values[i:j])` is exactly `j - i`, with no `+ 1` anywhere
- one number describes a boundary: `mid` is where the left piece ends and where the right piece
  begins, both at once
- adjacent ranges `[a, b)` and `[b, c)` tile a list with no gap and no overlap
- an empty range is expressible as `i == j`, which a closed range cannot do cleanly

A closed range gives you length `last - first + 1`, so every split needs both `mid` and `mid + 1`,
which is where off-by-one bugs live.

0-indexing is a separate choice. The reason it is the common default is that an index then equals the
number of elements before it, so index and count become the same kind of number. That is why
`values[i:j]` has `j - i` elements and `range(n)` runs exactly `n` times.

## How to think about the midpoint

Do not ask "which element is in the middle". Ask "where do I cut".

Picture the list as a ruler with marks at `0, 1, 2, ..., n`, with the elements sitting between the
marks. Indexing picks an element by its left mark. Slicing picks a span between two marks. The
midpoint is a mark, not an element, so there is no `n - 1` to write.

Checks that remove all doubt:

- the right piece starts exactly where the left piece ends, and both are `mid`
- the pieces add back up: `mid + (n - mid) = n`
- both pieces are strictly smaller than `n` whenever `n >= 2`

| n | cut at `n // 2` | left | right |
| --- | --- | --- | --- |
| 2 | 1 | 1 element | 1 element |
| 4 | 2 | 2 elements | 2 elements |
| 5 | 2 | 2 elements | 3 elements |

## One line for a Q&A

"We cut at `n // 2` rather than computing the middle element. The cut is a position between elements,
so the same number is where the left half ends and where the right half begins. That is why there is
no `+ 1` anywhere, and why the two halves always add back to `n`."
