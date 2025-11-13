Returns a list of Döner stores in Stuttgart based on Filters

| Parameter          | Type                 | Description                       | Allowed Values / Format                                                    |
| ------------------ | -------------------- | --------------------------------- | -------------------------------------------------------------------------- |
| `rating`           | number (1–5)         | Minimum overall rating            | 1,2,3,4,5                                                                  |
| `ai_rating`        | number (0–100)       | Minimum AI rating score           | 0–100                                                                      |
| `price_min`        | number (€)           | Minimum price                     | 0–50                                                                       |
| `price_max`        | number (€)           | Maximum price                     | 0–50                                                                       |
| `district`         | comma-separated list | Stuttgart districts               | Any valid Stuttgart district (see below)                                   |
| `distance_from_me` | number (m)           | Max distance from user location   | Any integer ≥ 0                                                            |
| `open_hours`       | comma-separated list | Filter by current open status     | `open`, `not_open`, `late_open`                                            |
| `vegetarian`       | comma-separated list | Filter by food type               | `meat`, `vegetarian`, `vegan`                                              |
| `halal`            | string               | Halal availability                | `halal`, `not_halal`                                                       |
| `doners_available` | comma-separated list | Döner items sold                  | `Falafel`, `Chicken`, `Lamb`, `Beef`, `Lahmacun`, `Yufka`, `Feta`, `Ayran` |
| `waiting_time`     | string               | Average waiting time              | `fast`, `normal`, `slow`                                                   |
| `payment`          | comma-separated list | Accepted payment types            | `cash`, `card`                                                             |
| `seating`          | comma-separated list | Seating options                   | `seating`, `to_go`                                                         |
| `sauces_available` | comma-separated list | Sauces offered                    | `Knoblauch`, `Kräuter`, `Joghurt`, `Scharf`                                |
| `limit`            | number               | Max results (default 20, max 100) | 1–100                                                                      |
| `offset`           | number               | Pagination offset                 | ≥ 0                                                                        |

### **All Possible Filter Values**

#### 🏙️ Districts (Stuttgart)

`Stuttgart-Mitte`, `Stuttgart-Nord`, `Stuttgart-Süd`, `Stuttgart-Ost`, `Stuttgart-West`,  
`Bad Cannstatt`, `Feuerbach`, `Zuffenhausen`, `Vaihingen`, `Möhringen`,  
`Degerloch`, `Plieningen`, `Sillenbuch`, `Hedelfingen`, `Wangen`,  
`Botnang`, `Birkach`, `Untertürkheim`, `Obertürkheim`, `Mühlhausen`, `Weilimdorf`

#### 🕒 Open Hours

- `open` → currently open
- `not_open` → currently closed
- `late_open` → open past 23:00

#### 🥗 Vegetarian Options

- `meat`
- `vegetarian`
- `vegan`

#### 💶 Payment Options

- `cash`
- `card`

#### 🍽️ Seating Options

- `seating`
- `to_go`

#### 🌯 Döner Types

- `Falafel`
- `Chicken`
- `Lamb`
- `Beef`
- `Lahmacun`
- `Yufka`
- `Feta`
- `Ayran`

#### 🌶️ Sauces

- `Knoblauch`
- `Kräuter`
- `Joghurt`
- `Scharf`

#### ⏱️ Waiting Times

- `fast`
- `normal`
- `slow`

### **Example Request**
```HTTP
GET /api/stores?district=Stuttgart-West&rating=4&ai_rating=80&price_min=3&price_max=7&open_hours=open,late_open&vegetarian=vegan,vegetarian&halal=halal&doners_available=Falafel,Chicken&waiting_time=fast&payment=cash,card&seating=seating,to_go&sauces_available=Knoblauch,Scharf&limit=10
```

### **Example Response**
```json
{
  "data": [
    {
      "id": "place_001",
      "name": "Dönerhaus West",
      "district": "Stuttgart-West",

      "location": {
        "coordinates": { "lat": 48.7761, "lng": 9.1653 },
        "google_place_id": "ChIJd0ener01",
        "address": "Rotebühlstraße 25, 70178 Stuttgart, Germany",
        "plus_code": "Q5VH+2G Stuttgart, Germany",
        "maps_url": "https://www.google.com/maps/place/?q=place_id:ChIJd0ener01"
      },

      "rating": 4.6,
      "ai_rating": 87,
      "price": 5.5,
      "vegetarian": ["vegetarian"],
      "halal": "halal",
      "waiting_time": "fast",
      "payment": ["cash", "card"],
      "seating": ["seating", "to_go"],
      "doners_available": ["Falafel", "Chicken", "Yufka"],
      "sauces_available": ["Knoblauch", "Scharf"],
      "open_hours": "open",
      "distance_from_me": 820,

      "ai_review": "Dönerhaus West offers fresh ingredients, flavorful chicken döner, and excellent vegetarian options. Customers praise the quick service and friendly staff.",

      "opening_hours": {
        "monday": "10:00–22:00",
        "tuesday": "10:00–22:00",
        "wednesday": "10:00–22:00",
        "thursday": "10:00–23:00",
        "friday": "10:00–23:00",
        "saturday": "11:00–23:00",
        "sunday": "11:00–21:00"
      },

      "reviews": [
        {
          "id": "rev_101",
          "user": "Max Mustermann",
          "date": "2025-11-10",
          "rating": 5,
          "text": "Best döner in Stuttgart! Fast service and very clean."
        },
        {
          "id": "rev_102",
          "user": "Anna Müller",
          "date": "2025-11-05",
          "rating": 4,
          "text": "Good taste, but sauce was a bit spicy for me."
        }
      ]
    }
  ],
  "meta": {
    "total": 1,
    "limit": 10,
    "offset": 0
  }
}

```