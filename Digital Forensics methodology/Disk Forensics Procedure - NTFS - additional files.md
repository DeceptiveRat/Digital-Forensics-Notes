``` py
# extract_artifacts.py
#!/bin/python3

import sys
import getopt
import glob
import pandas as pd
import os
import subprocess

def usage():
	print("usage:", sys.argv[0])
	print("options:")
	print("-h: display this help screen")
	print("-o <offset>: offset of FS")
	print("-f <filename>: filename (defaults to *MFTECmd*.csv)")
	print("-i <filename>: image file")
	print("-p <path>,<path>: paths to save result files")
	print("-a <artifact>,<artifact>,...: artifacts to extract. Available options: prefetch, lnk, ")

def get_rows(artifact, mft_df) -> pd.DataFrame:
	files_df = pd.DataFrame()
	if artifact == "prefetch":
		# prefetch conditions
		files_df = mft_df.loc[
			(mft_df['ParentPath'] == ".\\Windows\\Prefetch") & 
			(mft_df['FileName'].str.endswith('.pf')), 
			['EntryNumber', 'FileName']
		]
	elif artifact == "lnk":
		# lnk conditions
		return files_df
	else:
		return files_df
	
	return files_df

def main():
	try:
		opts, args = getopt.getopt(sys.argv[1:], "ho:f:i:p:a:")
	except getopt.GetoptError as err:
		print(err)
		usage()
		sys.exit(2)

	offset = -1
	filename = ""
	image = ""
	paths=[]
	artifacts = []

	for option, argument in opts:
		if option == "-h":
			usage()
			sys.exit()
		elif option == "-o":
			offset = int(argument)
		elif option == "-f":
			filename = argument
		elif option == "-i":
			image = argument
		elif option == "-p":
			paths = argument.split(",")
		elif option == "-a":
			artifacts = argument.split(",")
		else:
			assert False, "unhandled option"

	if offset == -1:
		print("Please enter offset of FS with -o")

	if filename == "":
		filename = "*MFTECmd*.csv"

	for file in glob.glob(filename):
		mft_df = pd.read_csv(file, low_memory=False)
		for artifact, path in zip(artifacts, paths):
			files_df = get_rows(artifact, mft_df)

			# check file df for error
			if files_df.empty:
				print("Enter correct artifact name")
				usage()
				exit(-1)
			
			# extract each file in df
			for row in files_df.itertuples(index=False):
				entry_number = row.EntryNumber
				file_name = row.FileName
				cmd = ["icat", "-o", str(offset), image, str(entry_number)]

				# prepend path 
				if path != "":
					os.makedirs(path, exist_ok=True)
					outpath = path+"/"+file_name
				else:
					outpath = file_name

				# run command
				with open(outpath, "wb") as outfile:
					subprocess.run(cmd, stdout=outfile, check=True)

if __name__ == "__main__":
	main()
```