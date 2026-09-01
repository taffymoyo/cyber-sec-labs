File path traversal, traversal sequences stripped non-recursively

Lab: File path traversal, traversal sequences stripped non-recursively

App strips the literal substring ../ from user input as a defense, but only does a single, non-recursive pass — removing a match doesn't trigger a re-scan of the result.

THE VULNERABILITY

filename=../../../etc/passwd           -> blocked (../ stripped, breaks the traversal)
filename=....//....//....//etc/passwd  -> 200 OK (contents of /etc/passwd returned!)

The filter scans for ../ and deletes it once. Feeding it ....// causes the filter to find and remove ../ from the middle of the string, but the leftover characters reassemble into a working ../ sequence — because the filter never re-checks its own output.

THE ATTACK

1. Intercept a request that fetches a file via a filename parameter, send to Repeater
2. Try plain traversal ../../../etc/passwd -> blocked, filter strips ../
3. Replace each ../ with ....// :
   filename=....//....//....//etc/passwd
4. Send -> filter strips one ../ out of each ....// chunk, leaving ../ behind each time
5. Reassembled path resolves to ../../../etc/passwd -> traversal succeeds

HOW ....// ACTUALLY WORKS

- Take ....// and look at the characters: . . . . / /
- The filter finds a literal ../ match sitting at characters 2-4 (. . /) and deletes just that chunk
- What's left stitched together: .. + / = ../
- One single-pass strip is not enough — the result of stripping still contains the banned pattern, but the filter never looks again

WHY IT'S VULNERABLE

- Filter does a single, non-recursive removal pass — it assumes removing a bad pattern once is sufficient
- Filter sanitizes (deletes/modifies) input instead of rejecting it outright
- No re-validation of the output of the sanitization step

PREVENTION

- Reject input containing traversal sequences outright, rather than trying to "clean" it by stripping
- If stripping is used, apply it in a loop until no more matches are found (fully recursive removal), or use a regex that itself accounts for overlapping/reconstructable patterns
- Validate the final, resolved path against the intended base directory instead of relying on pattern removal from the raw string
- Use a whitelist of allowed filenames/IDs rather than accepting raw paths from the client

WHAT I LEARNT — HOW THIS DIFFERS FROM DOUBLE ENCODING

This lab uses a different bypass mechanism than the double URL-encoding trick (%252e%252e%252f -> ../../../), even though both are "path traversal filter evasion":

                    | Double Encoding                                    | ....// trick
--------------------|-----------------------------------------------------|--------------------------------------------------
What's exploited    | Filter decodes fewer times than the backend does   | Filter removes bad patterns only once, doesn't re-check its own output
Characters involved | %XX percent-encoding                               | Plain literal text, no encoding at all
Filter's job        | Decode, then string-match                          | Strip/delete matched substrings ("sanitize")
Root cause          | Inconsistent number of decode passes across parts | Non-recursive sanitization

- Double encoding exploits a decode-count mismatch between the filter and the component that actually uses the value.
- ....// exploits non-recursive string removal — deleting a bad substring can accidentally construct a new instance of that same bad substring, and a single-pass filter never notices.
- Both fall under "the filter checks something naive/superficial instead of validating the true resolved value" — the same root lesson as the SSRF blacklist bypasses and the null-byte extension bypass: any filter that pattern-matches on raw/partially-processed input (instead of the fully resolved final value) can be defeated by a mismatch between what the filter sees and what the backend actually uses.
