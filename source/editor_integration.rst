
On OSX get logs in terminal with:

log stream --predicate 'eventMessage contains "s4m"' --info --style compact
- could later figure out how to filter out the fields we don't want with 
awk and regular expressions, skipping for now
