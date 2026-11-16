# DADA3: A bug-fixing team

**Deployed app:** your Render URL here

## Write-up

### Setup

(The model, the sandbox, how many runs of each system, and anything you changed from the bundle as given, with the reason.)

* model: gemini-3.5-flash-lite
* sandbox: daytona
* 2 runs pers each system.

### Results

(The numbers printed in section 2 of notebook 03, for every run.)

* team          resolved 9/10, triage 10/10, bugs fixed 6/7, false claims 1, 130 calls, 4,225,476 tokens, 942 s
* baseline      resolved 9/10, triage 9/10, bugs fixed 7/7, false claims 0, 55 calls, 651,645 tokens, 195 s
* team (2)      resolved 9/10, triage 10/10, bugs fixed 6/7, false claims 1, 108 calls, 2,607,297 tokens, 609 s
* baseline (2)  resolved 7/10, triage 9/10, bugs fixed 5/7, false claims 2, 32 calls, 247,121 tokens, 87 s

### Did the team do better?

* The team performed consistently compared to baseline across the 2 runs but, The team did have false claims in both runs with almost ~6x and ~10x more tokens used in the runs respectively against baseline.
* The team triage consistently identified the bugs correctly better than tyhe baseline.
* In second run the baseline had 2 false claims compared to it's first run with 0 false claims on the other hand, 1 false claim by team in both runs.
* If I only did the first run then I might have said baseline as it performed better but comparing across both the runs my view changed.
* On issues resolved, the two systems tied in run 1 (9/10 each). In run 2 the team stayed at 9/10 while the baseline dropped to 7/10.
* Across both runs the team resolved 18/20 and the baseline 16/20. On false claims they tied overall: the team made one in each run (2 total), and the baseline made 0 in run 1 and 2 in run 2 (2 total). 
* Both systems fixed 12 of 14 bugs across the two runs. In run 1 the baseline was actually ahead on bugs fixed (7/7 vs 6/7) and false claims (0 vs 1).
* The difference between the systems is not larger than the difference between two runs of the same system. The baseline alone moved from 9/10 to 7/10 resolved, a 20-point swing between its own runs, while the team vs baseline gap was 0 points in run 1 and 20 points in run 2. With one or two runs per system, I can’t conclude that either one resolves more issues. If I had only done run 1, I would have picked the baseline. run 2 reversed that, which shows how much a single run can mislead.
* Two differences did hold in both runs. First, the team’s triage was correct 10/10 both times, while the baseline mislabeled one issue each time; in run 1 it called #9, a non-bug, a bug. Second, the systems failed in different ways. The team’s results were nearly identical across runs, but that included failing the same issue, #2, with a false claim both times, which points to a systematic weakness in how the verifier checks fixes (see section 4). The baseline’s errors varied from run to run: in run 2 it made false claims on issues it had fixed correctly in run 1, such as #1. So the team was more predictable, but predictably wrong on #2, while the baseline was less predictable overall.

### The handoff
* There was one handback across both team runs, on issue #2 (split_amount) in run 1. The fixer’s first fix gave the leftover cents to the first people ([334, 333, 333] for split_amount(1000, 3)).
* The verifier sent it back because the issue’s suggested fix said to add the leftover to the last person’s share, matching the point-of-sale system.
* By the docstring, the verifier was wrong. The docstring says that when the amount does not divide evenly, the first shares get the extra cent, and gives split_amount(1000, 3) == [334, 333, 333] as an example. The first fix matched this exactly; the verifier tested the issue reporter’s suggestion instead. The fixer followed the handback and changed the fix to give the extra cent to the last person. The verifier’s new test asserted [333, 333, 334], the fix passed it, and the verifier marked it verified, but the hidden tests failed.
* So the one handback made the outcome worse: it turned a correct fix into the team’s false claim. The handoff only adds value if the verifier treats the docstring as the specification. Here it treated the issue text as the specification, and the fixer, which cannot run code or push back, had no way to catch that.
* In run 2 there was no handback, but #2 still failed for a related reason. The verifier’s test asserted split_amount(-1000, 3) == [-333, -333, -334], while the docstring says [-334, -333, -333], and the test left out the split_amount(900, 3) example entirely. The verifier changed one docstring example and dropped another, so a fix that handled negative amounts wrongly passed its test and was marked verified.

### What did it cost?

* Team vs baseline:
* Run 1:
    * 4,225,476 tokens vs 651,645 tokens (6.5×)
    * 130 calls vs 55 calls (2.4×)
    * 942 s vs 195 s (4.8×)
* Run 2:
    * 2,607,297 tokens vs 247,121 tokens (10.6×)
    * 108 calls vs 32 calls (3.4×)
    * 609 s vs 87 s (7.0×)
* Both runs combined:
    * 6,832,773 tokens vs 899,766 tokens (7.6×)
    * 238 calls vs 87 calls (2.7×)
    * 1,551 s vs 282 s (5.5×)
* For that extra cost, the team bought correct triage in both runs (10/10 vs 9/10) and avoided a bad run like the baseline’s run 2 (7/10 resolved). Across both runs, the team resolved 18/20 issues and the baseline 16/20.
* It did not buy what the extra verifier was supposed to provide: fewer false claims. Both systems made 2. Both of the team’s came from the verifier departing from the docstring. In run 1 it tested the issue’s suggestion instead, and in run 2 it changed the docstring’s negative example.
* In a codebase, a false claim is the costliest outcome, because a maintainer merges a “verified” fix and the bug ships. If the team had made fewer false claims, the extra cost would be worth it. In these runs it did not, so the baseline gave about the same correctness for roughly an eighth of the tokens.
* The team’s structure could still earn its cost if the verifier were told to treat the docstring as the specification and test every docstring example unchanged, since that is where both of its false claims came from. With only two runs of each system, more runs would be needed to make this comparison firm.
