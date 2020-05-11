# Mongo Geospatial Exercise

The coordinates will be in the format `[Longitude, Latitude]` as specified in the [GeoJSON specification](https://tools.ietf.org/html/rfc7946#section-3.1.1) and the [MongoDB documentation](https://docs.mongodb.com/manual/reference/geojson/)

The geometries are available in the data.geojson file.

**Insert 3 coordinates from Paris :**

```javascript
db.places.insertMany([
  {
    type: 'Feature',
    properties: {
      name: 'Eiffel Tower',
    },
    geometry: {
      type: 'Point',
      coordinates: [2.2979021072387695, 48.8560752090976],
    },
  },
  {
    type: 'Feature',
    properties: {
      name: 'Arc de Triomphe',
    },
    geometry: {
      type: 'Point',
      coordinates: [2.2950375080108643, 48.873783275868725],
    },
  },
  {
    type: 'Feature',
    properties: {
      name: 'Notre-Dame de Paris',
    },
    geometry: {
      type: 'Point',
      coordinates: [2.3500120639801025, 48.8529832421452],
    },
  },
]);
```

```bash
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5eb9555bc79f9e95b243dcf0"),
		ObjectId("5eb9555bc79f9e95b243dcf1"),
		ObjectId("5eb9555bc79f9e95b243dcf2")
	]
}
```

**Create a 2dsphere index on the GeoJSON objects :**

```js
db.places.createIndex({ geometry: '2dsphere' });
```

```bash
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
```

**Get the nearest point from a random point in Paris :**

```javascript
db.places
  .find({
    geometry: {
      $near: {
        $geometry: {
          type: 'Point',
          coordinates: [2.33583927154541, 48.86101631231847],
        },
      },
    },
  })
  .limit(1);
```

```javascript
{
  "_id" : ObjectId("5eb9555bc79f9e95b243dcf2"),
  "type" : "Feature",
  "properties" : { "name" : "Notre-Dame de Paris" },
  "geometry" : {
    "type" : "Point",
    "coordinates" : [ 2.3500120639801025, 48.8529832421452 ]
  }
}
```

**Insert a polygon including 2 of the previously created points :**

```javascript
db.places.insertOne({
  type: 'Feature',
  properties: {},
  geometry: {
    type: 'Polygon',
    coordinates: [
      [
        [2.2936534881591797, 48.878772022014644],
        [2.2807788848876953, 48.85302559914698],
        [2.3205184936523438, 48.85167015731634],
        [2.2936534881591797, 48.878772022014644],
      ],
    ],
  },
});
```

```bash
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5eb956f2c79f9e95b243dcf4")
}
```

**Check if the polygon include the points :**

```js
db.places.find({
  $and: [
    (geometry: {
      $geoWithin: {
        $geometry: {
          type: 'Polygon',
          coordinates: [
            [
              [2.2936534881591797, 48.878772022014644],
              [2.2807788848876953, 48.85302559914698],
              [2.3205184936523438, 48.85167015731634],
              [2.2936534881591797, 48.878772022014644]
            ]
          ],
        },
      },
    }),
    { geometry: { type: 'Point' } },
  ],
});
```

```bash
{
  "_id" : ObjectId("5eb9555bc79f9e95b243dcf1"),
  "type" : "Feature", "properties" : { "name" : "Arc de Triomphe" },
  "geometry" : {
    "type" : "Point",
    "coordinates" : [ 2.2950375080108643, 48.873783275868725 ]
  }
}
{
  "_id" : ObjectId("5eb9555bc79f9e95b243dcf0"),
  "type" : "Feature", "properties" : { "name" : "Eiffel Tower" }, "geometry" : {
    "type" : "Point",
    "coordinates" : [ 2.2979021072387695, 48.8560752090976 ]
  }
}
```

**Find all the point included in the radius of a point :**

This request show all the point in the radius of 1 km of the Châtelet subway station. The query converts the distance to radians by dividing by the approximate equatorial radius of the earth: 6371 km.

```javascript
db.places.find({
  geometry: {
    $geoWithin: {
      $centerSphere: [[2.3473620414733887, 48.85881405227578], 1 / 6371],
    },
  },
});
```

```bash
{
  "_id" : ObjectId("5eb9555bc79f9e95b243dcf2"),
  "type" : "Feature", "properties" : { "name" : "Notre-Dame de Paris" }, "geometry" : {
    "type" : "Point",
    "coordinates" : [ 2.3500120639801025, 48.8529832421452 ]
  }
}
```
