import requests
from flask import Flask, jsonify, request

app = Flask(__name__)

API_TOKEN = ""
RSA_KEY = ""


class InvalidUsage(Exception):
    status_code = 400

    def __init__(self, message, status_code=None, payload=None):
        Exception.__init__(self)
        self.message = message
        if status_code is not None:
            self.status_code = status_code
        self.payload = payload

    def to_dict(self):
        rv = dict(self.payload or ())
        rv["message"] = self.message
        return rv


@app.errorhandler(InvalidUsage)
def handle_invalid_usage(error):
    response = jsonify(error.to_dict())
    response.status_code = error.status_code
    return response


@app.route("/")
def home_page():
    return "<p><h2>Weather API: python Saas.</h2></p>"


@app.route("/content/api/v1/integration/generate", methods=["POST"])
def weather_endpoint():
    json_data = request.get_json()

    if json_data.get("token") is None:
        raise InvalidUsage("Token is required", status_code=400)

    token = json_data.get("token")

    if token != API_TOKEN:
        raise InvalidUsage("Wrong API token", status_code=403)


    city = json_data.get("city")
    if city is None:
        raise InvalidUsage("City parameter is required", status_code=400)

    date = json_data.get("date")
    if date is None:
        raise InvalidUsage("Date parameter is required", status_code=400)


    start_datetime = f"{date}T00:00:00"
    end_datetime = f"{date}T23:59:59"

    weather = get_weather(city, start_datetime, end_datetime)

    values = weather["locations"][f"{city}"]["values"]

    weather_inf = []

    for value in values:
        datetime_str = value["datetimeStr"]
        humidity = value["humidity"]
        temperature = value["temp"]
        wind_speed = value["wspd"]
        cloud_cover = value["cloudcover"]

        weather_inf.append({
            "datetime": datetime_str,
            "humidity": humidity,
            "temperature": temperature,
            "wind_speed": wind_speed,
            "cloud_cover": cloud_cover,
        })

    result = {
        "requester_name": json_data.get("requester_name"),
        "data": json_data.get("date"),
        "city": json_data.get("city"),
        "weather_info": weather_inf,
    }

    return result


def get_weather(city, start_datetime, end_datetime):
    api_url = f"https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/weatherdata/history?aggregateHours=24&startDateTime={start_datetime}&endDateTime={end_datetime}&unitGroup=metric&contentType=json&location={city}&key={RSA_KEY}"

    headers = {"X-Api-Key": RSA_KEY}

    response = requests.get(api_url, headers=headers)

    if response.status_code == requests.codes.ok:
        return response.json()
    else:
        raise InvalidUsage(response.text, status_code=response.status_code)

