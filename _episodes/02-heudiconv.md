---
title: "Converting DICOM data to BIDS"
teaching: 5
exercises: 15
questions:
- "How do I convert my data to BIDS?"
objectives:
- "Understand that different software have different requirements"
- "Learn to create a heuristic file for your study"
keypoints:
- "Automate conversion of DICOM data to BIDS"
- "The specification is still the key source of information"
---

## What is Heudiconv?
[Heudiconv](https://github.com/nipy/heudiconv) is a **heuristic based** dicom converter.

## Install Heudiconv and it's requirements (or use docker or singularity or vagrant or ...)?

If you have docker then do: `docker pull nipy/heudiconv`

Otherwise you will need:

- miniconda
- dcm2niix
- heudiconv
- git annex (for getting dicoms)

let's organize some dicomdata.

```
mkdir data
cd data
git clone http://datasets.datalad.org/test/dartmouth-siemens/PHANTOM1_3/.git
cd PHANTOM1_3
git annex get YAROSLAV_DBIC-TEST1
cd ..
curl -O https://raw.githubusercontent.com/nipy/heudiconv/master/heuristics/convertall.py
```

Now let's convert the data:

```
docker run --rm -it -v $PWD:/data nipy/heudiconv -d /data/%s/YAROSLAV_DBIC-TEST1/HEAD_ADVANCED_APPLICATIONS_LIBRARIES_20160824_104430_780000/*/*IMA -s PHANTOM1_3 -f /data/convertall.py -c dcm2niix -b -o /data/output
```

## Create your heuristic file?

First, let's take a look at the dicominfo.txt file created.

Now let's see the [README](https://github.com/nipy/heudiconv/blob/master/README.md)

Let's consider converting only the t1 and the resting series

- First create keys
- Then use the information from dicominfo.txt to fill those keys.

~~~
t1 = create_key('anat/sub-{subject}_T1w')
rest = create_key('func/sub-{subject}_dir-{acq}_task-rest_run-{item:02d}_bold')

info = {t1: [], rest: []}
last_run = len(seqinfo)
for s in seqinfo:
    print(s)
    # TODO: clean it up -- unused stuff laying around
    x, y, sl, nt = (s[6], s[7], s[8], s[9])
    if (sl == 176) and (nt == 1) and ('t1' in s[12]):
        info[t1] = [s[2]]
    if (nt > 10) and ('taskrest' in s[12]):
        if s[13]:
            info[rest].append({'item': s[2], 'acq': 'corrected'})
return info
~~~
{: .python}


```
docker run --rm -it -v $PWD:/data nipy/heudiconv -d /data/%s/YAROSLAV_DBIC-TEST1/HEAD_ADVANCED_APPLICATIONS_LIBRARIES_20160824_104430_780000/*/*IMA -s PHANTOM1_3 -f /data/heuristic_2type.py -c dcm2niix -b -o /data/output
```
