#  Track the evolution of Stockfish mate finding effectiveness 

Track the performance of [official Stockfish](https://github.com/official-stockfish/Stockfish)
in finding the (best) mates within the 6554 mate problems in [`matetrack.epd`](matetrack.epd).
The raw data is available in [`matetrack1000000.csv`](matetrack1000000.csv),
and is visualized in the graphs below.

<p align="center">
  <img src="matetrack1000000all.png?raw=true">
</p>

<p align="center">
  <img src="matetrack1000000.png?raw=true">
</p>

---

### Usage of `matecheck.py`

```
usage: matecheck.py [-h] [--engine ENGINE] [--nodes NODES] [--depth DEPTH] [--time TIME] [--mate MATE] [--hash HASH] [--threads THREADS] [--syzygyPath SYZYGYPATH] [--minTBscore MINTBSCORE] [--maxTBscore MAXTBSCORE] [--concurrency CONCURRENCY] [--engineOpts ENGINEOPTS] [--epdFile EPDFILE [EPDFILE ...]] [--showAllIssues] [--shortTBPVonly] [--showAllStats] [--bench]

Check how many (best) mates an engine finds in e.g. matetrack.epd, a file with lines of the form "FEN bm #X;".

options:
  -h, --help            show this help message and exit
  --engine ENGINE       name of the engine binary (default: ./stockfish)
  --nodes NODES         nodes limit per position, default: 10**6 without other limits, otherwise None (default: None)
  --depth DEPTH         depth limit per position (default: None)
  --time TIME           time limit (in seconds) per position (default: None)
  --mate MATE           mate limit per position: a value of 0 will use bm #X as the limit, a positive value (in the absence of other limits) means only elegible positions will be analysed (default: None)
  --hash HASH           hash table size in MB (default: None)
  --threads THREADS     number of threads per position (values > 1 may lead to non-deterministic results) (default: None)
  --syzygyPath SYZYGYPATH
                        path(s) to syzygy EGTBs, with ':'/';' as separator on Linux/Windows (default: None)
  --minTBscore MINTBSCORE
                        lowest cp score for a TB win (default: 19754)
  --maxTBscore MAXTBSCORE
                        highest cp score for a TB win: if nonzero, it is assumed that (MAXTBSCORE - |score|) is distance in plies to first zeroing move in(to) TB (default: 20000)
  --concurrency CONCURRENCY
                        total number of threads script may use, default: cpu_count() (default: 32)
  --engineOpts ENGINEOPTS
                        json encoded dictionary of generic options, e.g. tuning parameters, to be used to initialize the engine (default: None)
  --epdFile EPDFILE [EPDFILE ...]
                        file(s) containing the positions and their mate scores (default: ['matetrack.epd'])
  --showAllIssues       show all unique UCI info lines with an issue, by default show for each FEN only the first occurrence of each possible type of issue (default: False)
  --shortTBPVonly       for TB win scores, only consider short PVs an issue (default: False)
  --showAllStats        show nodes and depth statistics for best mates found (always True if --mate is supplied) (default: False)
  --bench               provide cumulative statistics for nodes searched and time used (default: False)
```

Sample output:
```
Using ./sf16 on matetrack.epd with --nodes 10000
Engine ID:     Stockfish 16
Total FENs:    6554
Found mates:   524
Best mates:    355

Parsing the engine's full UCI output, the following issues were detected:
Bad PVs:       600   (from 351 FENs)
```

Note that the mate counts are out of the total number of FENs, while the
reported issues cover all the UCI output lines received from the engine.
Here a "bad" PV may mean that it is too short, too long, allows a draw,
contains illegal moves or does not end in checkmate.

### List of available test suites

* `ChestUCI_23102018.epd`: The original suite derived from publicly available `ChestUCI.epd` files, see [FishCooking](https://groups.google.com/g/fishcooking/c/lh1jTS4U9LU/m/zrvoYQZUCQAJ). It contains 6566 positions, with one definite and five likely draws, some illegal positions and some positions with a sub-optimal or likely incorrect value for the fastest known mate.
* **`matetrack.epd`**: The successor to `ChestUCI_23102018.epd`, with all illegal positions removed and all known errors corrected. The plots shown above are based on this file. It contains 6554 mate problems, ranging from mate in 1 (#1) to #126 for positions with between 4 and 32 pieces. In 26 positions the side to move is going to get mated.
* `matetrackpv.epd`: The same as `matetrack.epd`, but for each position the file also includes a PV leading to the checkmate, if such a PV is known.
* `matedtrack.epd`: Derived from `matetrackpv.epd` by applying a best move in all those positions, where the winning side is to move, and where a best move is known. The order of the positions in `matedtrack.epd` corresponds 1:1 to the order in `matetrack.epd`. So the new test suite still contains 6554 mate problems, but for 6547 of them the side to move is going to get mated. Observe that due to duplications, only 6529 of the latter positions are unique.
* `mates2000.epd`: A smaller test suite with 2000 positions ranging from #1 to #27. It contains a random selection of positions from `matetrack.epd` and `matedtrack.epd` that Stockfish can solve with 1M nodes. In 1105 positions the side to move is going to get mated.
* `cursed.epd`: A collection of 125 cursed wins and 189 cursed losses, where wins and losses are denoted by `bm #1` and `bm #-1', respectively.

### Automatic creation of new test positions

With the help of the script `advancepvs.py` it is easy to derive new mate
puzzles from the information stored in `matetrackpv.epd`. For example, the file `matedtrack.epd` has been created with the command
```shell
python advancepvs.py --plies 1 --mateType won && sed 's/; PV.*/;/' matedtrackpv.epd > matedtrack.epd
```
Similarly, a file with only `#-10` positions can be created with the command
```shell
python advancepvs.py --targetMate -10 && grep 'bm #-10;' matedtrackpv.epd | awk -F'; PV' '\!seen[$1]++' > mate-10.epd
```
