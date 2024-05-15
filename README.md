import tkinter as tk
import requests
from tkinter import messagebox
from PIL import Image, ImageTk
import ttkbootstrap

API_KEY = "992a04c803eff7ae34ab713685ea94e7"

def get_weather(city):
    url = f"https://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}"
    try:
        response = requests.get(url)
        response.raise_for_status()
        weather_data = response.json()
        icon_id = weather_data['weather'][0]['icon']
        temperature = weather_data['main']['temp'] - 273.15
        description = weather_data['weather'][0]['description']
        location = f"{weather_data['name']}, {weather_data['sys']['country']}"
        icon_url = f"http://openweathermap.org/img/wn/{icon_id}@2x.png"
        return icon_url, temperature, description, location
    except requests.exceptions.RequestException as e:
        messagebox.showerror("Error", f"Failed to fetch weather data: {e}")
        return None

def search():
    city = city_entry.get().strip()
    if not city:
        messagebox.showwarning("Warning", "Please enter a city name.")
        return

    result = get_weather(city)
    if result:
        icon_url, temperature, description, location = result
        location_label.config(text=location)
        try:
            icon_response = requests.get(icon_url, stream=True)
            icon_response.raise_for_status()
            weather_icon = ImageTk.PhotoImage(Image.open(icon_response.raw))
            icon_label.configure(image=weather_icon)
            icon_label.image = weather_icon
        except Exception as e:
            messagebox.showerror("Error", f"Failed to fetch weather icon: {e}")
        temperature_label.config(text=f"Temperature: {temperature:.2f}°C")
        description_label.config(text=f"Description: {description}")

root = ttkbootstrap.Window(themename="morph")
root.title("Weather App")
root.geometry("400x400")

tk.Label(root, text="Please enter a city:", font="Helvetica, 18").pack(pady=10)
city_entry = ttkbootstrap.Entry(root, font="Helvetica, 18")
city_entry.pack(pady=10)

ttkbootstrap.Button(root, text="Search", command=search, bootstyle="warning").pack(pady=10)

location_label = tk.Label(root, font="Helvetica, 25")
location_label.pack(pady=20)

icon_label = tk.Label(root)
icon_label.pack()

temperature_label = tk.Label(root, font="Helvetica, 20")
temperature_label.pack()

description_label = tk.Label(root, font="Helvetica, 20")
description_label.pack()

root.mainloop()
