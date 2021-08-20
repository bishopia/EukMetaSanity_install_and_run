I couldn't get EMS install.sh to work just right, so i followed the below instead.


## Setup miniconda

install fresh miniconda version

```
cd ~
sh Miniconda3-latest-MacOSX-x86_64
```


## Install EMS part 1: conda environments

open VNC desktop client (2core 7days). this is easier than ssh because if things take a long time the terminal won't time out.

make new work directory and clone EMS
```
cd ~/data/ibishop/software
mkdir EMS
cd EMS
git clone https://github.com/cjneely10/EukMetaSanity.git
cd EukMetaSanity
```

check out INSTALL.sh and start the conda environment construction process, using mamba not conda
```
cat INSTALL.sh 
#create new env
mamba create --prefix /users/ibishop/data/ibishop/condas/EukMS_run

#install dependencies into new env
mamba env update --prefix /users/ibishop/data/ibishop/condas/EukMS_run --file /gpfs/home/ibishop/data/ibishop/software/EMS/EukMetaSanity/bin/run-pipeline/run/environment.yml

#load new env
conda activate /users/ibishop/data/ibishop/condas/EukMS_run

#install pip stuff
python -m pip install .

#deactivate conda env
conda activate
```

repeat with other two conda envs (refine and report)


## Install EMS part 2: outside dependencies and bug fixes

eggnog-mapper install
```
#load conda env
conda activate /gpfs/home/ibishop/data/ibishop/condas/EukMS_report

#install eggnog-mapper via pip
/gpfs/home/ibishop/data/ibishop/condas/EukMS_report/bin/pip install eggnog-mapper

#deactivate
conda activate
```

kofascan install (instructions are here: https://www.genome.jp/ftp/tools/kofam_scan/INSTALL)
```
cd ~/data/ibishop/software
mkdir -p ./kofamscan/db
cd ./kofamscan/db
wget ftp://ftp.genome.jp/pub/db/kofam/ko_list.gz 
wget ftp://ftp.genome.jp/pub/db/kofam/profiles.tar.gz 
gunzip ko_list.gz 
tar xvzf profiles.tar.gz 
cd ../..
mkdir -p ../kofamscan/bin
cd kofamscan/bin
wget ftp://ftp.genome.jp/pub/tools/kofamscan/kofamscan.tar.gz 
tar xvzf kofamscan.tar.gz 

mkdir kofamscan/src
cd kofamscan/src 
tar xvjf parallel-latest.tar.bz2 
cd parallel-20190322 
./configure --prefix=$HOME/kofamscan/parallel 
make 
make install 
```

now add new Dfam file and unzip and configure repeatmasker
```
this is the absolute path you need when configuring RepeatMasker:
/gpfs/home/ibishop/data/ibishop/condas/EukMS_run/bin
```

then fix augustus bug
```
cd /gpfs/data/epscor/ibishop/condas/EukMS_run/bin
sed -i 's/transcript_id \"(\.\*)\"/transcript_id \"(\\S\+)"/' filterGenesIn_mRNAname.pl
cd /gpfs/data/epscor/ibishop/condas/EukMS_refine/bin
sed -i 's/transcript_id \"(\.\*)\"/transcript_id \"(\\S\+)"/' filterGenesIn_mRNAname.pl
```

then download databases (mmetsp and odb10). make sure the run config files can find these databases. i put it in scratch but i might move it over to ~/data/ibishop
```
interact -t 2:00:00 -n 10 -m 100G
cd ~/scratch/2ems_data
module load python/3.9
conda activate /gpfs/home/ibishop/data/ibishop/condas/EukMS_run

#run download-data, making sure to use the python3 version in EukMW_run conda environment
/gpfs/home/ibishop/data/ibishop/condas/EukMS_run/bin/python3 ~/data/ibishop/software/EMS/EukMetaSanity/EukMetaSanity/download-data -t 10 -d .```
```

you should be good now, try the test data.
