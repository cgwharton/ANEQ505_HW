**Load data
```
cp - r /pl/active/courses/2025_summer/CSU_2025/raw_reads_oxycow .
```

 **Launch a session and load qiime**
```
ainteractive --ntasks=6 --time=02:00:00
module purge
module load qiime2/2024.10_amplicon
```

**Create directories for the different analyses:**
```
mkdir slurm

mkdir taxonomy

mkdir tree

mkdir taxaplots

mkdir dada2

mkdir demux

mkdir core_metrics
```
