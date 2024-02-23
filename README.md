![Weather Pi](https://github.com/sourceduty/Weather_Pi/assets/123030236/929df30c-55f9-4900-bdc4-49cf898da7b9)

Conceptual plan for a Raspberry Pi project that involves collecting local weather data using sensors, comparing this data with publicly available weather data, and generating a comparison report. There are numerous Raspberry Pi projects focused on reading and logging weather data, but this particular project is exclusive to Sourceduty.

#
### Hardware Components

1. Raspberry Pi (any model with Wi-Fi capability, e.g., Raspberry Pi 3/4)
2. DHT22, DHT11 or BME280 sensor (for temperature and humidity readings)
3. BMP180 or BMP280 sensor (for barometric pressure readings)
4. Real Time Clock (RTC) module (e.g., DS3231 for accurate time keeping)
5. MicroSD card (for OS installation and data storage)
6. Power supply for Raspberry Pi (powerbank)
7. Portable solar panel
8. 3D printed weatherproof enclosure for outdoor sensor deployment

#
### Software and Libraries

1. Raspberry Pi OS (previously called Raspbian)
2. Python 3 (programming language)
3. Libraries for sensor communication:
   - Adafruit_DHT library for DHT22 sensor
   - Adafruit_BME280 library for BME280 sensor
   - Adafruit_BMP280 library for BMP280 sensor
   - Adafruit_CircuitPython_DS3231 for the DS3231 RTC module
4. SQLite or any lightweight database for local data storage
5. Flask (Python web framework) for creating the comparison report page
6. Requests library for fetching published weather data from the internet

#
### Concept Code

```
import Adafruit_DHT
import Adafruit_BMP280
import requests
import datetime
import sqlite3
from flask import Flask, render_template

# Initialize sensors and global variables
DHT_SENSOR = Adafruit_DHT.DHT22
DHT_PIN = 4  # Assuming the DHT22 data pin is connected to GPIO4
BMP_SENSOR = Adafruit_BMP280.BMP280()
RTC_MODULE = None  # Replace with RTC module setup if necessary
LOCATION = "YourLocation"  # Manually set, can be made dynamic

# Setup database connection
conn = sqlite3.connect('weather_data.db')
c = conn.cursor()

# Create table for local weather data
c.execute('''CREATE TABLE IF NOT EXISTS local_weather
             (date text, time text, temperature real, humidity real, pressure real)''')

# Function to read from sensors
def read_sensors():
    humidity, temperature = Adafruit_DHT.read_retry(DHT_SENSOR, DHT_PIN)
    pressure = BMP_SENSOR.read_pressure()
    now = datetime.datetime.now()
    date = now.strftime("%Y-%m-%d")
    time = now.strftime("%H:%M:%S")
    return date, time, temperature, humidity, pressure

# Function to save data to database
def save_data(date, time, temperature, humidity, pressure):
    c.execute("INSERT INTO local_weather VALUES (?, ?, ?, ?, ?)", (date, time, temperature, humidity, pressure))
    conn.commit()

# Function to fetch public weather data
def fetch_public_data():
    # Use an API like OpenWeatherMap (You'll need an API key)
    # Replace 'YourAPIKey' with your actual API key and 'YourCityID' with your city's ID
    response = requests.get("http://api.openweathermap.org/data/2.5/weather?id=YourCityID&appid=YourAPIKey&units=metric")
    data = response.json()
    pub_temperature = data['main']['temp']
    pub_humidity = data['main']['humidity']
    pub_pressure = data['main']['pressure']
    return pub_temperature, pub_humidity, pub_pressure

# Function to compare local and public data
def compare_data():
    # Fetch last recorded local data
    c.execute("SELECT * FROM local_weather ORDER BY date DESC, time DESC LIMIT 1")
    local_data = c.fetchone()
    # Fetch public data
    pub_temperature, pub_humidity, pub_pressure = fetch_public_data()
    # Compare and prepare report data
    report = {
        'local_temperature': local_data[2],
        'local_humidity': local_data[3],
        'local_pressure': local_data[4],
        'public_temperature': pub_temperature,
        'public_humidity': pub_humidity,
        'public_pressure': pub_pressure
    }
    return report

# Flask app to display comparison report
app = Flask(__name__)

@app.route('/')
def report_page():
    report = compare_data()
    return render_template('report.html', report=report)

if __name__ == '__main__':
    # Example to read and save data periodically (implement scheduling)
    date, time, temperature, humidity, pressure = read_sensors()
    save_data(date, time, temperature, humidity, pressure)
    
    # Start Flask app
    app.run(host='0.0.0.0', port=8080)
```

![Box Concept](https://github.com/sourceduty/Weather_Pi/assets/123030236/055118e7-12ed-46c3-8a2d-9632b812023e)

***

🛈 This software is free and open-source; anyone can redistribute it and/or modify it.
