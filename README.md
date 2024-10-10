# øving_6
gruppearbeid 

import csv
import matplotlib.pyplot as plt
from datetime import datetime

# Initialize lists
dato = []
tid = []
barometer = []
trykk = []
temperatur = []
lufttemperatur = []
Lufttrykk_i_havnivå = []



# Function to parse dates with multiple formats
def parse_date(date_str):
    formats = ["%m.%d.%Y %H:%M", "%d.%m.%Y %H:%M"]
    for fmt in formats:
        try:
            return datetime.strptime(date_str, fmt)
        except ValueError:
            continue
    return None  # Return None if no format matches

# Function to read a CSV file and process its data
def read_csv_file(file_path):
    data = []
    with open(file_path, mode="r") as file:
        df = csv.DictReader(file, delimiter=";")
        for row in df:
            data.append(row)
    return data

# Read and process first CSV file
file1 = '/Users/fahmi/.spyder-py3/Fk/trykk_og_temperaturlogg_rune_time.csv.txt'
data1 = read_csv_file(file1)

for row in data1:
    try:
        date_obj = parse_date(row["Dato og tid"])
        if date_obj:
            dato.append(date_obj)
            tid.append(float(row["Tid siden start (sek)"]))
            # Replace commas with dots and convert to float
            if row["Trykk - barometer (bar)"]:
                barometer.append(float(row["Trykk - barometer (bar)"].replace(',', '.')))
            if row["Trykk - absolutt trykk maaler (bar)"]:
                trykk.append(float(row["Trykk - absolutt trykk maaler (bar)"].replace(',', '.')))
            if row["Temperatur (gr Celsius)"]:
                temperatur.append(float(row["Temperatur (gr Celsius)"].replace(',', '.')))
    except ValueError as e:
        print(f"Error converting row: {row}, error: {e}")

luft_temperatur = []

# Read and process second CSV file
file2 = '/Users/fahmi/.spyder-py3/Fk/temperatur_trykk_met_samme_rune_time_datasett.csv.txt'
data2 = read_csv_file(file2)
print(data2[0].keys())  # Print column names for verification
for row in data2:
    try:
        lufttemperatur.append(float(row["Lufttemperatur"].replace(',', '.')))
    except ValueError as e:
        print(f"Error converting row: {row}, error: {e}")
# finne gjennomsnitt 
def calculate_averages(times, temperatures, n):
    valid_times = []
    averages = []
    
    for i in range(n, len(temperatures) - n):
        window_temperatures = temperatures[i - n:i + n + 1]
        average_temp = sum(window_temperatures) / len(window_temperatures)
        averages.append(average_temp)
        valid_times.append(times[i])
    
    return valid_times, averages


times = dato
temperatures = temperatur

# Calculate averages for n=30
n = 30
valid_times, averages = calculate_averages(times, temperatures, n)

# Plotting
plt.figure(figsize=(10,4))
plt.plot(times, temperatures, label='Temperatur', color='blue')
plt.plot(valid_times, averages, label='Gjennomsnittsverdi (n=30)', color='orange')
plt.plot(times[:len(lufttemperatur)], lufttemperatur, label='Lufttemperatur', color='green')
plt.xlabel('Tid')
plt.ylabel('Temperatur')
plt.legend()
plt.show()

#plotting2

plt.figure(figsize=(10,4))
plt.plot(times, trykk) 
plt.plot()
plt.plot()
plt.show()

