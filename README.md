# Eye Cipher Field Notes

A plain-English running log of an attempt to make progress on Noita's unsolved "eye messages",
plus a SAT-based solver for "shared-shuffle" deck ciphers (a form of XGAK).

- Report: https://frobthebuilder.github.io/eye-cipher-field-notes/
- Solver: [`solver/xgak_sat.py`](solver/xgak_sat.py) (single file, Python 3.9+, `pip install python-sat`).
  See [`solver/README.txt`](solver/README.txt) for the cipher model, the options and the caveats.

The eyes are **not** decoded. The solver has been tested only on fake messages. The report and the
code were written by an AI (Claude) working with the repository owner.
