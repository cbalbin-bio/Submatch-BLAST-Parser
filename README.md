If using outside of this module import the module.
```python
from blastparser import BLASTParser
```

Instantiate BLASTParser

```python
bp = BLASTParser("path/to/query/seq", "path/to/blast/db")
```
Alternatively, to filter blast results using taxids, pass a taxid or a list of taxids to exclude from blast search. This requires that you have get_species_taxids.sh in your path and you used a taxid map when creating your db.

get_species_taxids.sh is part of blast+

For example the call below will exlude all coronaviruses
```python
bp = BLASTParser("path/to/query/seq", "path/to/blast/db", taxids=11118)
```
<br>
Retrieve the parsed hits.

This returns an instance of HitsDataContainer which is essentially just a list with
some convience methods I wrote for filtering. The hits list contains instances of HitData.
```python
hits = bp.gethits()
```

<br>
Filters out any hits not containing 5 consecutive matches somewhere in the hit.

```python
hits= hits.filterbymatchlen(5)
```

You can chain filters together.
```python
hits = hits.filterbymatchlen(5).filterbyident(.60).filterbycover(60)
```


Only printing first 3 hits for this example.
```python
for hit in hits[:3]:
    # you can access the properties of the HitData for each
    print(f"Subject accession {hit.subject_accession}")
    print(" Query Match range(s):", "".join([f"{match_range[0]}-{match_range[1]}, " for match_range in hit.match_ranges]), "  Match len(s):", ", ".join([f"{match_len}" for match_len in hit.match_lengths]))
    print()
    print(f"Query  {hit.query_start:<5d} {hit.aln_query_seq} {hit.query_end:<5d}")
    print(f"            ", "".join(hit.cigar))
    print(f"Subj   {hit.subject_start:<5d} {hit.aln_subject_seq} {hit.subject_end:<5d}")
    print()


    # to handle subbhits (mutliple consecutive hits of >= 5 in a single blast hit)
    # remember hit.matches ranges is a lists of lists as structured as below
    # [[start_pos_int, end_pos_int], [start_pos_int, end_pos_int], ...]
    for index, (start, end) in enumerate(hit.match_ranges):

        # subtract 1 from start to go back to 0 based index (blast uses 1 based index) 
        if end - (start-1) >=5:
            #only print subhits greater than 5
            print(f"{start} {hit.submatch_seqs[index]} {end}")



    print()
    print()
    print()
    print()
```