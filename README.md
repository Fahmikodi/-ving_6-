import csv
import matplotlib.pyplot as plt
from datetime import datetime
import numpy as np

# Initialize lists
dato = []
tid = []
tid_2 = []
barometer = []
trykk = []
temperatur = []
lufttemperatur = []

# Function to parse dates with multiple formats
def parse_date(date_str):
    formats = ["%m.%d.%Y %H:%M", "%d.%m.%Y %H:%M"]
    for fmt in formats:
        try:
            return datetime.strptime(date_str, fmt)
        except ValueError:
            continue
    return None  

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
            if row["Trykk - barometer (bar)"]:
                barometer.append(float(row["Trykk - barometer (bar)"].replace(',', '.')))
            if row["Trykk - absolutt trykk maaler (bar)"]:
                trykk.append(float(row["Trykk - absolutt trykk maaler (bar)"].replace(',', '.')))
            if row["Temperatur (gr Celsius)"]:
                temperatur.append(float(row["Temperatur (gr Celsius)"].replace(',', '.')))
    except ValueError as e:
        print(f"Error converting row: {row}, error: {e}")

# Read and process second CSV file
file2 = '/Users/fahmi/.spyder-py3/Fk/temperatur_trykk_met_samme_rune_time_datasett.csv.txt'
data2 = read_csv_file(file2)

for row in data2:
    date_obj = parse_date(row["Tid(norsk normaltid)"])
    if date_obj:
        tid_2.append(date_obj)  # Store the datetime object instead of converting to float
        lufttemperatur.append(float(row["Lufttemperatur"].replace(',', '.')))
        
        
plt.figure(figsize=(10, 4))
plt.plot(dato, temperatur, label='Temperatur (File 1)', color='blue')
#plt.plot(tid_2, lufttemperatur, label='Lufttemperatur (File 2)', color='green')
plt.xlabel('Tid')
plt.ylabel('Temperatur')
plt.legend()
plt.show()

plt.figure(figsize=(10,4))
plt.hist(temperatur, bins=range(int(min(temperatur)), int(max(temperatur)) + 1), alpha=0.5, label='Temperatur (File 1)')
plt.hist(lufttemperatur, bins=range(int(min(lufttemperatur)), int(max(lufttemperatur)) + 1), alpha=0.5, label='Lufttemperatur (File 2)')
plt.xlabel('Temperatur')
plt.ylabel('Frequency')
plt.legend()
plt.show()
        
pressure_diff = [abs_p - bar_p for abs_p, bar_p in zip(trykk, barometer) if bar_p is not None]

# Calculate moving average
def moving_average(data, n=10):
    return [sum(data[i-n:i+n+1]) / (2*n+1) for i in range(n, len(data)-n)]

avg_pressure_diff = moving_average(pressure_diff)

# Plotting
plt.figure(figsize=(10,4))
plt.plot(dato[:len(pressure_diff)], pressure_diff, label='Pressure Difference', color='red')
plt.plot(dato[:len(avg_pressure_diff)], avg_pressure_diff, label='Average Pressure Difference', color='orange')
plt.xlabel('Tid')
plt.ylabel('Pressure Difference')
plt.legend()
plt.show()        
        
   # Read and process weather station data  fix her 
file3 = '/Users/fahmi/.spyder-py3/Fk/temperatur_trykk_sauda_sinnes_samme_tidsperiode.csv.txt'
data3 = read_csv_file(file3)

sinnes_data = [row for row in data3 if row["Stasjon"] == "Sinnes"]
sauda_data = [row for row in data3 if row["Stasjon"] == "Sauda"]

# Extract data for plotting
sinnes_dates = [parse_date(row["Tid(norsk normaltid)"]) for row in sinnes_data]
sinnes_temps = [float(row["Lufttemperatur"].replace(',', '.')) for row in sinnes_data]

sauda_dates = [parse_date(row["Tid(norsk normaltid)"]) for row in sauda_data]
sauda_temps = [float(row["Lufttemperatur"].replace(',', '.')) for row in sauda_data]

# Plotting
plt.figure(figsize=(10,4))
plt.plot(sinnes_dates, sinnes_temps, label='Sinnes', color='blue')
plt.plot(sauda_dates, sauda_temps, label='Sauda', color='green')
plt.xlabel('Tid')
plt.ylabel('Lufttemperatur')
plt.legend()
plt.show()     
        
        # Calculate differences
temp_diff = [t1 - t2 for t1, t2 in zip(temperatur, lufttemperatur)]
pressure_diff = [p1 - p2 for p1, p2 in zip(trykk, barometer)]

# Find min and max differences
min_temp_diff = min(temp_diff)
max_temp_diff = max(temp_diff)
min_pressure_diff = min(pressure_diff)
max_pressure_diff = max(pressure_diff)

print(f"Min temperature difference: {min_temp_diff}")
print(f"Max temperature difference: {max_temp_diff}")
print(f"Min pressure difference: {min_pressure_diff}")
print(f"Max pressure difference: {max_pressure_diff}")
        
        # Calculate standard deviation
def calculate_standard_deviation(data, n=30):
    mean = np.mean(data)
    std_dev = np.std(data)
    return mean, std_dev

mean_temp, std_dev_temp = calculate_standard_deviation(temperatur)

# Plotting with error bars
plt.figure(figsize=(10,4))
plt.errorbar(dato, temperatur, yerr=std_dev_temp, errorevery=30, capsize=5, label='Temperatur with Std Dev', color='blue')
plt.xlabel('Tid')
plt.ylabel('Temperatur')
plt.legend()
plt.show()
        
        
