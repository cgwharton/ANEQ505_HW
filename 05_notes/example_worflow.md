### Project summary 
I like to include a little about what I am working on. 

#### Working directory 
```
/scratch/alpine/USER/aneq505
```

#### Class reservation 
```
sinteractive --reservation=aneq505 --time=01:00:00 --partition=amilan --nodes=1 --ntasks=2 --qos=normal
```

#### Out of class reservation 
```
ainteractive --time=01:00:00 --parition=amilan --notes=1 --ntasks=2 --qos=normal
```

- Time can be adjusted based on how long you think you will need the reservation for.. 

### Load Qiime2/ Qiime2 version
- qiime2 analysis was done with qiime2 amplicon version 2024.10
```
module purge
module load qiime2/2024.10_amplicon
```



I then like to put the date I worked on something so for example :

2/1/2026
### Demultiplexing 
```
command for demultiplexing
```

- I will then put a note for how everything went ex: 
	- Demultiplexing completed
	- Average quality score from demux summary was 30. 
	- Will trim at 0 bp.
	- Will truncate at 250 bp. 
- I will also save the qiime2 .qzv files from analysis and save them in the same file as my obsidian vault and then link the files below the command (you don't have to this, I do so it's easy to find them later if I want to reference them).