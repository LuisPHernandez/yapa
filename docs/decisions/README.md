# Decision records

One short markdown file per important decision. 

## When to write one

Write a record when a choice is hard to reverse, or when a future teammate (or our future selves) would ask "why did they do it this way?".

Don't write one for things the code already makes obvious, or for choices that are cheap to change.

## How

1. Copy `0000-template.md` to `NNNN-short-title.md`, using the next free number (`0001-payment-gateway.md`).
2. Fill it in. Half a page is plenty.
3. Open a PR. The review is where the other person agrees or pushes back; merging it means the decision is made.

Records are never edited to say something different. If we change our minds, write a new record and set the old one's status to `Superseded by NNNN`.
