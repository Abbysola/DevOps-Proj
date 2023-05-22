Create the project folder called Shell
mkdir Shell

Move into the Shell folder
cd Shell

In the Shell directory, create a file for your Shell script named “script_task.sh”
touch script_task.sh

Copy/write your Shell script into the *”script_task.sh” file using vim editor.
vi script_task.sh

NOTE: You can develop your shell script on a note pad, before copying and paste into the “script_task.sh” file.

Make the script file executable:
chmod +x script_task.sh

Run the script :
sudo ./script_task.sh

#!/bin/bash

# 1. Ask the user for their name and age, and output a message with their name and the year they were born

# Name
echo "What is your name?"
read name

# Age
echo "What is your age?"
read age

# Birth Year
current_year=$(date +%Y)
birth_year=$((current_year - age))

# Display the user's name and  birth year
echo "My name is $name. I was born in $birth_year"

# 2. Create a new directory with a name provided by the user, and navigate into it.

# Ask for the directory name
echo "Enter a name for the directory:"
read dir_name

# Create the directory
mkdir "$dir_name"

# Navigate into the directory
cd "$dir_name"

# Display the current working directory
echo "You are now in: $(pwd)"

# 3. List all files in the current directory, sorted by file size

ls -lS

# 4. Count the number of files in the current directory
file_count=$(ls -1 | wc -l)

# Output the result
echo "Number of files in the current directory: $file_count"

# 5. Take a list of numbers as input from the user and output the sum of those numbers.
echo "Enter a list of numbers separated by spaces:"
read numbers

# Calculate the sum of numbers
sum=0
for num in $numbers; do
  sum=$((sum + num))
done

# Output the result
echo "Sum of the numbers: $sum"

# 6. Output a random number between 1 and 100
random_number=$((RANDOM % 100 + 1))

# Output the random number
echo "Random number: $random_number" 

# 7. Create a backup of a specified file by copying it to a backup directory with a timestamp in the filename.
# Specify the file to be backed up
original_file="/Users/abisolaajuwon/Desktop/DareyPractice/Project4/Port3300.png"

# Create a timestamp for the backup filename
timestamp=$(date +%Y%m%d%H%M%S)

# Backup directory
backup_dir="/Users/abisolaajuwon/Desktop/BackupDareyPractice"

# Create the backup file with a timestamp in the filename
backup_file="${backup_dir}file_${timestamp}.txt"
cp "$original_file" "$backup_file"

# Output the backup file path
echo "Backup created: $backup_file"

# 8. Check if a website is online and output a message indicating whether it is up or down.

# Specify the website URL
website="https://www.linkedin.com/"

# Send a GET request and store the response code
response=$(curl -Is "$website" | head -n 1 | awk '{print $2}')

# Check the response code
if [[ $response =~ ^[2-3][0-9]{2}$ ]]; then
  echo "The website $website is up."
else
  echo "The website $website is down."
fi

# 9. Convert a temperature in Celsius to Fahrenheit, using input from the user:
# Ask the user for the temperature in Celsius
echo "Enter the temperature in Celsius:"
read celsius

# Convert Celsius to Fahrenheit
fahrenheit=$(echo "scale=2; $celsius * 9/5 + 32" | bc)

# Output the temperature in Fahrenheit
echo "Temperature in Fahrenheit: $fahrenheit"

# 10. Ask the user for a sentence, then output the sentence in reverse order. For example, if the user enters “Hello, world!”, the script should output “!dlrow ,olleH”

# Ask the user for a sentence
echo "Enter a sentence:"
read sentence

# Reverse the sentence using a loop
reversed_sentence=""
for ((i=${#sentence}-1; i>=0; i--))
do
    reversed_sentence="${reversed_sentence}${sentence:$i:1}"
done

# Output the reversed sentence
echo "Reversed sentence: $reversed_sentence"
