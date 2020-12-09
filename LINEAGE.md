# Lineage — fw/v4.1-mc-refactor

★ SHRAVAN LAL — the gold PID refactor. 40 commits exclusive to master, Dec 2020.
Authored as "Shravan Lal <Shravan Lal>" (real developer; distinct from Bradley's <Shravan Lal> email identity-bug).
Architecture: cascaded dual-loop PID (position → velocity → duty), per-axis feedforward, first-tick derivative suppression, dt-scaled integration, modular `src/refactor/{pid,follower,positioner,movement,preloader,math_util,parser,generic_cmd}.{c,h}`.
Status: UNFINISHED — his own commit 7d28ac3 says so. PID output never wired to motors. Last on-hardware test 2020-10-21.
Effort to finish: 18-27 engineer-days.
Upstream: https://bitbucket.org/whatboxcnc/cnc-firmware-v4.1 branch mc-refactor @ 7ed6058
