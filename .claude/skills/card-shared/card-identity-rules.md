# Card Identity Rules

## Goal

Match the user’s input to one exact card variant before returning any facts.

## Rules

- Parse issuer, family, and variant separately when possible.
- Do not merge facts across similarly named variants.
- Both personal and business credit cards are supported. If a card exists in both personal and business versions and the user doesn't specify, treat it as ambiguous.
- If multiple plausible variants exist, name the leading match and list the competing variants in `confidence_notes`.
- If the exact match cannot be determined, return the best-supported variant with `status: unconfirmed` in the relevant summary field.

## Examples

- `Chase Sapphire` is ambiguous between Preferred and Reserve.
- `Amex Gold` is ambiguous between American Express Gold Card (personal) and American Express Business Gold Card.
- `Ink Preferred` or `CIP` resolves confidently to Chase Ink Business Preferred.
