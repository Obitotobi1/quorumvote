# QuorumVote

A poll where GenLayer validators grade every free-text answer against the poll's rubric, and only answers that pass are counted.

Ordinary polls count votes. QuorumVote counts answers that meet a stated standard: "does this proposal name a concrete safeguard?", "does this reply address the stated risk?". Validators judge each answer, and the contract keeps a PASS or FAIL only when the validators' quote really appears in the answer.

## Deployment (GenLayer Studio)

Contract: `0x73aa64bC93223289Be4d0375Dfc86FBBb2f21C93`


Explorer: https://explorer-studio.genlayer.com/address/0x73aa64bC93223289Be4d0375Dfc86FBBb2f21C93

## How it works

1. `create_poll(question, rubric)` - anyone opens a poll (question 8-200 chars, rubric 8-300 chars).
2. `submit_answer(poll_id, text)` - one answer per wallet (8-400 chars), no identical duplicates, up to 20 answers per poll.
3. `evaluate(poll_id, index)` - anyone asks validators to grade one answer. Each validator runs the model; consensus uses `gl.eq_principle.prompt_comparative`, so the `verdict` field must match.
4. `close_poll(poll_id)` - only the poll owner.
5. `get_tally(poll_id)` - counts only graded answers: passed, failed, pending and the indexes that passed.

## Safety

- The model's output is not trusted. A PASS or FAIL is kept only if its quote appears word for word in the answer (checked in code). Otherwise the answer stays PENDING and can be retried, up to 3 attempts.
- Graded answers are final.
- Question, rubric and answer are each fenced in their own tag with `<` and `>` escaped, so text cannot close a tag or add instructions.
- Limits: 20 polls per wallet, 20 answers per poll.

## Read API for other contracts

`get_poll`, `get_tally`, `get_polls_by_owner`, `get_recent`, `passed_count`.

## Tests

13 local tests, no node needed: `python3 tests/test_quorumvote.py`

They cover validation and limits, one answer per wallet, a PASS with a real quote, a FAIL not counting as a pass, an invented quote staying pending, garbage output, the attempt limit, final grading, owner-only close, full polls, prompt fencing (including a hostile answer) and the views.

## Live test (Studio, validators on)

| Step | Result |
|---|---|
| create_poll 1 and 2 (DAO treasury question, rubric "name a concrete safeguard") | FINALIZED |
| Poll 1, answer "Require a multisig with a timelock ..." then evaluate | get_tally: passed 1, failed 0, pending 0, passing_indexes [0] |
| Poll 2, answer "Just trust the team and hope ..." then evaluate | get_tally: passed 0, failed 1, pending 0 |
| submit_answer with poll_id -1 | rejected, nothing stored |

## Files

- `quorumvote.py` - the Intelligent Contract
- `tests/test_quorumvote.py` - local tests
