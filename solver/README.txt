Class-XGAK known-plaintext / crib solver for the Noita eye messages
====================================================================

AI-written research code (Claude, Oct 2026), part of the "Eye Cipher Field Notes" report.
Tested only on fake messages; it has NOT decoded the eyes. Use at your own risk.

What it does
------------
It recovers the key of a "shared-shuffle" deck cipher (a form of XGAK) when the plaintext,
or just its letter pattern, is known. Or it proves that no such key exists for a given guess.

The cipher model (read-then-shuffle form):

    83-card deck D. Each plaintext letter a has a read position k_a (all different) and
    belongs to a group (class) c(a). For each letter:
        output the card D[k_a]
        then shuffle the deck with the group's shuffle s_c:  D_new[p] = D[s_c[p]]
    Letters in the same group share one shuffle and differ only in which card they read.
    All messages start from the same deck.
    No doubles <=> a card just read never lands on a read position.

This is equivalent to "shuffle, then read position o_a" with k_a = s_c(o_a). With one group
per letter it is the community's XGAK; ordinary GAK (the deck cipher) is the special case
k_a = s_a(0).

The trick: whether two symbols t and t+d of a message are equal depends only on the
plaintext letters between them, not on the deck before. So the problem splits into many
small constraints:

    "this sequence of group shuffles must (or must not) bring this card to that read spot"

A SAT solver (CaDiCaL via python-sat) handles those well. Positions can be relabelled freely,
so WLOG k_a = a. Cards of different messages are tied together through the shared starting deck.

The solver only needs:
  * which positions share a letter (the actual letters don't matter);
  * optionally, which group each letter is in. With K=2 groups it can find the grouping itself.

Install
-------
    Python 3.9 or newer
    pip install python-sat

xgak_sat.py is a single file. The eye messages and a ~25 KB Finnish text sample (used for fake
messages) are built in. Save it as xgak_sat.py; nothing else is needed. crib_example.txt shows
the crib format, and eyes_ascii32.txt is the eye data for reference (value = ord(char) - 32).

Usage
-----
    python xgak_sat.py -h
    python xgak_sat.py synth -h
    python xgak_sat.py eyes -h

1) Experiments on fake messages. Nine messages with the eyes' lengths, Finnish-like text folded
   onto A letters, a random key, 2 groups. The solver gets the ciphertext plus the plaintext.
   Success means the recovered key reproduces every symbol match and non-match in the full
   messages.

    python xgak_sat.py synth 23 6                     # 23-card deck, 6 letters
    python xgak_sat.py synth 41 10 --dmax 14
    python xgak_sat.py synth 53 14 --long             # --long: also use long-distance repeats
    python xgak_sat.py synth 23 6 --prefix 20         # only the first 20 letters of each message known
    python xgak_sat.py synth 23 6 --unknown-classes   # solver also works out the letter groups
    python xgak_sat.py synth 23 6 --mode mut0.02      # 2% of letters wrong: expect UNSAT

2) Testing a guess (crib) against the real eyes:

    python xgak_sat.py eyes crib_example.txt
    python xgak_sat.py eyes mycrib.txt --classes "0123456789|ABCDEFGHIJKLMNOPQRSTUVWXYZ "
    python xgak_sat.py eyes mycrib.txt --skip-label   # crib starts at the 2nd symbol

   Crib file: one line per message, "NAME: text", with NAME one of
   E1 W1 E2 W2 E3 W3 E4 W4 E5. The text is the guessed plaintext from the first symbol on
   (or from the second, with --skip-label). The same character means the same hidden letter,
   and spaces count as letters. Lines may differ in length. Leave out messages you have no
   guess for. Lines starting with # are comments.

   The result is CONSISTENT or IMPOSSIBLE under the model above.
   IMPORTANT: short cribs are easy to satisfy. CONSISTENT only means something once guesses
   of the same shape that are wrong (e.g. the same crib with letters shuffled, or real text
   from a book) come back IMPOSSIBLE. Always run such controls.

Results so far (fake messages, 2 groups, eye lengths, one key each)
-------------------------------------------------------------------
    deck  letters  time      result
     17      6     <1 s      exact key
     23      6     3-15 s    exact key
     31      8     49 s      exact key
     41     10     ~1 min    exact key
     53     14     3.5 min   exact key   (--long + symmetry breaking; 20 min with neither)
     61     16     25 min    exact key   (--long)
     83     22     running; at the current growth rate it may take days

    Crib only (first P letters of each message known):
      23 cards, P=20: exact
      41 cards, P=30: exact (49 s)
      53 cards, P=40: exact (200 s)
    Wrong plaintext, or 2% of letters changed: UNSAT in under a second.

Limitations
-----------
* It only applies if the eyes really are this kind of cipher. That is a hypothesis: it fits the
  statistics checked in the report, but it is not proven.
* It assumes distinct read positions (unique decryption) and one shared starting deck.
  Unknown groups are supported for K=2 only.
* Speed grows quickly with deck size. Whether 83 cards is practical is still being measured.
* --timeout kills the process, because CaDiCaL ignores interrupts.
* It does not yet decrypt beyond the crib, or search for cribs itself.
