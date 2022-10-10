---
layout: post
title: "Synchronize file from ROSMAP"
subtitle: "synaspeclient and synapseutils"
date: 2022-10-10 11:38:43
header-style: text
catalog: true
author: "Yuan"
tags: [Python, ROSMAP, Omics, Big data, Synpase]
---
{% include linksref.html %}
> 大丈夫处世，不能立功建业，不几与草木同腐乎？

Get accessing to data hold by [Synpase](https://www.synapse.org/#!Synapse:syn21314550) is easy as long as you have the permission. Below is my script to download data from there, using Python scripts.<br/>
I get all Syn_numbers for all files I need to download and save them in bamFiles.txt, and set that as the third parameter of the inputs.<br/>
Enjoy it if you also found useful.<br/>

{{note}}
The script is tested in 2022 and works fine.
{{end}}


```python
import synapseclient
import os
import pandas as pd
import sys
import datetime
 
#Read the bamFiles.txt to get all syn_numbers and the corresponding names
#filename = '~/public/ROSMAP/RNASeq/bamFiles.txt'
#filenumber = int(sys.argv[1])
start_num = int(sys.argv[1])
end_num = int(sys.argv[2])
#filename = ''.join(['./bamFilesBatch', filenumber])
filename = str(sys.argv[3])

df = pd.read_csv(filename,sep='\t',header=(0))
allsyn = df['synID']
totalLength=len(allsyn)
#Set the login information
syn = synapseclient.Synapse()
syn.login('UserName','Password')
 
# Obtain a pointer and download the data
# tmpsyn = ('syn4212586','syn4212972','syn4212973')
# Easier test
#tmpsyn= ('syn3505724','syn4300315','syn4300317')
#tmpsyn = allsyn[start_num:(end_num+1)]
newFolder = df['path'][start_num]#''.join((str(start_num),"_",str(end_num)))
directory_current = os.path.join(os.getcwd(),newFolder)
if not os.path.exists(directory_current):
	os.makedirs(directory_current)
totalNumber = end_num-start_num+1
currentNumber = start_num
for currentNumber in range(start_num,end_num+1): #Processing from start_num, to end_num
	syn_number = allsyn[currentNumber]
	print('starting:', (currentNumber+1),"/",totalLength,"(currentJobSize:",totalNumber,")\n", syn_number, "...\n")
	syn_pointer = syn.get(entity=syn_number,downloadLocation=newFolder)
	# Get the path to the local copy of the data file
	# filepath = syn_pointer.path 
	os.system('say "one file has finished"')
	print(syn_number,":",syn_pointer.path ,"\t Finished! \n")
	print(datetime.datetime.now(),"\n\n")
	syn_pointer = ()
```
---
