cat c1aa77aa_samples.lst | awk 'NR>=100 && NR<=200 {split($0,a," "); print a[1]}'
