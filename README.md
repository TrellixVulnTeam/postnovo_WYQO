# Postnovo
Post-processing peptide de novo sequences to improve their accuracy

Journal article: https://pubs.acs.org/doi/10.1021/acs.jproteome.8b00278

## Quick Start

Much more detail and other options can be found in the [Postnovo Wiki pages](https://github.com/semiller10/postnovo/wiki).

1. Postnovo runs on a Unix-like OS. Use a powerful server or desktop with plenty of available disk space (~25 GB).
2. Download and decompress the latest [release](https://github.com/semiller10/postnovo/releases).
3. Use Python 3.
[The Anaconda distribution](https://www.anaconda.com/distribution/) comes with all necessary package dependencies.
4. Download [DeNovoGUI](http://compomics.github.io/projects/denovogui.html) and large pre-trained models with Postnovo `setup`.

   For low-res MS2 data (example uses `nohup` to avoid termination upon logout and `&` to run in background):
   
   `nohup python main.py setup --denovogui --postnovo_low --deepnovo_low &`
   
   For high-res MS2 data:
   
   `python main.py setup --denovogui --postnovo_high --deepnovo_high`
   
5. Use the [ProteoWizard](http://proteowizard.sourceforge.net/) msconvert tool to convert your RAW file to an MGF file with a certain spectrum header format.

   `msconvert preformatted_spectra.raw --mgf --filter "titleMaker Run: <RunId>, Index: <Index>, Scan: <ScanNumber>"`

   Add additional filters as needed, here for peak picking and removal of zero intensity peaks.
   
   `msconvert preformatted_spectra.raw --mgf --filter "peakPicking vendor" --filter "zeroSamples removeExtra" --filter "titleMaker Run: <RunId>, Index: <Index>, Scan: <ScanNumber>"`

6. [Reformat the input MGF file](https://github.com/semiller10/postnovo/wiki/MGF-Input-File-Setup) for compatability with all de novo sequencing tools.

   `python main.py format_mgf --mgfs /path/to/preformatted_spectra.mgf --out /path/to/spectra.mgf`
   
7. [Set up a container with TensorFlow](https://github.com/semiller10/postnovo/wiki/DeepNovo-Installation) to run DeepNovo (optional but recommended for Postnovo). Superuser privileges may be required.

   `singularity build tensorflow.simg docker://tensorflow/tensorflow:latest`
8. Generate [DeepNovo](https://github.com/nh2tran/DeepNovo) de novo sequences (can take up to ~12 hours). The following examples consider low-res MS2 spectra. Processing high-res spectra requires more memory (see [link](https://github.com/semiller10/postnovo/wiki/Training-and-Running-DeepNovo)). Postnovo only supports standard fixed C and variable M PTMs at the moment.

   Using single machine with 32 cores:

   `python main.py predict_deepnovo --mgf /path/to/spectra.mgf --container /path/to/tensorflow.simg --frag_resolution low --cpus 32`
   
   Using compute cluster via Slurm with 16 cores per node and sufficient memory allocation:
   
   `python main.py predict_deepnovo --mgf /path/to/spectra.mgf --container /path/to/tensorflow.simg --frag_resolution low --cpus 16 --slurm --partition partition_with_16GB_mem --time_limit 36`
9. Download and run [MaRaCluster](https://github.com/statisticalbiotechnology/maracluster) (from the link) to cluster spectra by peptide species. As input, use the reformatted MGF file. Set "log10(p-value) threshold" to -2.

10. [Run Postnovo](https://github.com/semiller10/postnovo/wiki/Predicting-with-Postnovo), generating [Novor](https://www.rapidnovor.com/download/) and [PepNovo+](http://proteomics.ucsd.edu/Software/PepNovo/) de novo sequences via [DeNovoGUI](http://compomics.github.io/projects/denovogui.html) (can take up to ~3 hours). Results are written by default to the directory of the MGF input.

    `python main.py predict --mgf /path/to/spectra.mgf --clusters /path/to/MaRaCluster.clusters_p2.tsv --frag_method CID --   frag_resolution low --denovogui --deepnovo --cpus 32`
