TPrime Helper
==============
Python wrapper for TPrime, a C++ command line application for mapping times between data streams collected by SpikeGLX. Each stream must have a common sync signal, and the edge times from that sync signal must be extracted by CatGT. See the README for TPrime for details about parameters.

Dependencies
------------
[TPrime](https://billkarsh.github.io/SpikeGLX/)

Running
-------
```
python -m ecephys_spike_sorting.modules.TPrime_helper--input_json <path to input json> --output_json <path to output json>
```
Two arguments must be included:
1. The location of an existing file in JSON format containing a list of paths and parameters.
2. The location to write a file in JSON format containing information generated by the module while it was run.

See the `_schemas.py` file for detailed information about the contents of the input JSON.

TPrime Helper parses toStream_sync_params to find the SYNC file, assuming the data has been saved and processed through CatGT using probe folders. So, a file of sync edges from a probe is expected to be in the probe folder:

\catgt_<run name>_g<gate>\<run name>_g<gate>_imec<probe index>

spike_times.npy files for all probes for which there are sync edge files are read in and translated to seconds using the sample rate stored in params.py. (Note: the sample rate written out by rezToPhy is replaced by a value with more significant figures in the kilosort_helper module).

The CatGT command line is parsed to find all extracted edge files:

-All SY extracted (except toStream) are designated fromstreams

-For all probes not specified as the toStream, the file <probe_folder>\imecN_ks2\spike_times_sec.txt are designated events files

-If tostream is not XA or XD, niStream_sync is designated a fromstream

-All XA and XD extracted which are not specified as the the toStream or the niStream_sync are designated events files

These parameters are used to build the TPrime command line.

TPrime outputs the corrected times in text files. After it is run, these output times are resaved in npy format, as floats.

Notes:
- Since auxiliary data is usually collected at lower sample rates, a probe should be selected as the reference tostream

- Stated above, but worth repeating: this helper module assumes data stored in the probe folder format, with phy output written to <probe_folder>\imecN_ks2

Parameters
----------
- sync_period : in seconds, = 1.0 for sync generated by NP basestation
- toStream_sync_params : specify sync edges for tostream
- niStream_sync_params : specify sync edges for ni stream 

Input data
----------
- **CatGT extracted edge files for sync and events** : text files generated by CatGT with sync event edge times, in sec.
- **spike_times.npy files** : for each probe

Output data
-----------
- **spike_times_adj.txt, spike_times_adj.npy** : text files of spike times for each from stream