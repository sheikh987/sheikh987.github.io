If you want to learn how to represent data with colors using google maps, this blog is for you.




I had to make an app that uses colors to represent data on differnet districs in a state.
And I couldn't find a way to extarct a certain state from google maps(I had to use google maps for reasons that is not part of this blog). 
Also, I couldn't find a way to get a list of latitude and longitude for a particular district. so, I used openSteetMaps to get the latLng data.



After reading this blog you would be able to; 

* Extract data from openStreetMaps
* Use latLngs to create polygons
* Represent data in the form of colors


Lets use some python.

Create a file called **_extract.py_**

Copy the code below into **_extract.py_** and save it,


```python

import requests
import json
r = requests.get('http://polygons.openstreetmap.fr/get_poly.py?id={}&params=0'.format(1942920))
ccs = [{"lat" : i.split('\t')[1:][1], "long": i.split('\t')[1:][0]} for i in r.text.split('\n')[2:-3]]
json.dumps(ccs)

with open('test.json', 'w') as f:
     f.write(json.dumps(ccs))

```

open [openStreetmaps](https://www.openstreetmap.org).

Type the name of the state or district you want to search for and look for the **_Id of Relation_**

Open [polygons](http://polygons.openstreetmap.fr/index.py).

Paste the _Id_ and click _submit_.

Find the _poly_ column and click on _poly_.

You will get a list of _latLngs_ that make the chosen state or district.

You can also click on the _image_ column to check if you got the correct state or district shape

Coming back to the **_extract.py_** file we had created earlier,
We use the **_Id of realtion_** in the below line,

```python

r = requests.get('http://polygons.openstreetmap.fr/get_poly.py?id={}&params=0'.format(1942920))

```

You could change the **_Id of relation_** for differnt locations.

**_extract.py_** would create a _json_ file that you can save in **_test.json_** as shown below

```python

with open('test.json', 'w') as f:
     f.write(json.dumps(ccs))

```

**Note**: Dont forget to change the file name you are writing the json file into, everytime you change the _Id of relation_, or you will end up **overriding** _test.json_ 

Now, go to your teminal,

Type _python3_
Once you are in the python environment, run _extarct.py_ with the below command,

```
python3 extarct.py

```

and then you can look for **_test.json_**

### Creating Polygons

Import the _json_ files you have extracted into your android studio project **_raw_** folder.

Create an array of districts as shown below

```java

String[] arrayOfDistricts = {"jaipur", "udaipur", "kota"};

```

Now, create two _arrayList_ of type _Integer_ as shown below

```java

ArrayList<Integer> listOfDistricts = new ArrayList<Integer>();
ArrayList<Integer> listOfColors = new ArrayList<Integer>();

```
Now we can add the Json file in the _listOfDistricts_ and add colors in the _listOfColors_ as shown below

```java

listOfDistricts.add(R.raw.jaipur);
listOfDistricts.add(R.raw.udaipur);
listOfDistricts.add(R.raw.kota);

listOfColors.add(Color.RED);
listOfColors.add(Color.BLUE);
listOfColors.add(Color.GREEN);


```

Lets write a function to parse the _json_ file that we had imported in the _raw_ folder 

```java

    private ArrayList<LatLng> readItems(int resource) throws JSONException {
        ArrayList<LatLng> list = new ArrayList<LatLng>();
        InputStream inputStream = getResources().openRawResource(resource);
        String json = new Scanner(inputStream).useDelimiter("\\A").next();
        JSONArray array = new JSONArray(json);
        for (int i = 0; i < array.length(); i++) {
            JSONObject object = array.getJSONObject(i);
            double lat = object.getDouble("lat");
            double lng = object.getDouble("long");
            list.add(new LatLng(lat, lng));
        }
        Log.i("list",list.toString());
        return list;
    }
  
```

create a _HashMap<String, Polygon>_ where String is the name of the district and Polygon is the polygon generated

```java

HashMap<String ,Polygon> mapOfPolygons = new HashMap<String, Polygon>();

```

Also we need to create a GoogleMap object to which we will later add _polygons_ 

```java

GoogleMap m_map;

```

Lets write a function to create polygon from the above information

```java

    private void createPolygon(){
        ArrayList<LatLng> list = new ArrayList<LatLng>();
   
        try {
            for (int i=0; i<listOfDistricts.size(); i++) {
                list = readItems(listOfDistricts.get(i));
                Polygon polygon = m_map.addPolygon(new PolygonOptions().addAll(list)
                        .strokeColor(Color.TRANSPARENT)
                        .fillColor(listOfColors.get(i)));

                mapOfPolygons.put(arrayOfDistricts[i], polygon);

            }

            Log.i("list2", list.toString());
        } catch (JSONException e) {
            e.printStackTrace();
        }


    }
    
```    
### Represent data in the form of colors

we will generate random data for differnet data values. 

create a _HashMap<String, Integer>_ where String is the name of the district and Integer is the integer being generated randomly

```java

HashMap<String ,Integer> values = new HashMap<String ,Integer>();

```
 
 we can use the below code to generte random data and generate colors depending on the data generated
 
 ```java
 
                 for (int i=0; i<arrayOfDistricts.length; i++) {
                    values.put(arrayOfDistricts[i], (int)(Math.random() * 100));
                }

                for (HashMap.Entry<String ,Integer> entry : values.entrySet()) {
                    Integer x = entry.getValue();
                    if (x>=0 && x<=20) {
                        Polygon polygon = mapOfPolygons.get(entry.getKey());
                        polygon.setFillColor(Color.GRAY);

                    } else if ((x>20) && (x<=40)) {
                        Polygon polygon = mapOfPolygons.get(entry.getKey());
                        polygon.setFillColor(Color.YELLOW);

                    } else if ((x>40) && (x<=60)) {
                        Polygon polygon = mapOfPolygons.get(entry.getKey());
                        polygon.setFillColor(Color.GREEN);

                    } else if ((x>60) && (x<=80)) {
                        Polygon polygon = mapOfPolygons.get(entry.getKey());
                        polygon.setFillColor(Color.BLUE);

                    } else if ((x>80) && (x<=100)) {
                        Polygon polygon = mapOfPolygons.get(entry.getKey());
                        polygon.setFillColor(Color.RED);

                    }
                }
                
```                
we can use a button to trigger the above code

check this github [repository](https://github.com/sheikh987/Maps/tree/polygons)
