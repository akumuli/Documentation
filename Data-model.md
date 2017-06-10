# Data model

Akumuli’s data model is designed to track properties of real-world objects. These properties are supposed to be real-valued. Things like temperature and humidity measured over time by the sensors in the room are good examples of such properties. Another example is a CPU utilization and idle time on the virtual server.

# Tracking objects

Each object is uniquely addressed by the set of tags. For example, the room in the hotel can be pointed by the building and the room numbers (room=42 building=2). So, you can pin down specific room using only two tags.

The tags can be viewed as an address of the object, but they doesn’t limited to that. You can add redundant tags to increase the search capabilities. In our previous example redundant information like the floor number and the wing of the building (room=42 building=2 floor=1 wing=East) can be added to allow us to select all rooms in some specific part of the building. Akumuli’s query system allows you to use only the subset of tags to select the object (if this subset is unique for each object), e.g. you can still pin down the specific room using two tags (“room” and “building”) even if you have the “floor” and “wing” tags.
This means that you don’t need to create some unique ids or GUIDs to track your objects (but you still can add them as tags if you really want).

# Properties

Property is an aspect of the object that can be measured. For example, you can measure temperature in the room, voltage in the electric circuit, and amount of free space on disk. Each property has a name usually referred as a measurement name, e.g. ‘temperature’, ‘voltage’, ‘hddfree’. Measurement names should be unique within object. For example, if you want to measure amount of carbon monoxide in the room using two sensors you should use two different measurement names for both sensors. Otherwise you won’t be able to distinguish between measurements of both sensors:

    colevel_A room=42 building=2 floor=1 wing=East room_type=Luxe
    colevel_B room=42 building=2 floor=1 wing=East room_type=Luxe

The other option is to have a tag (but in this case the series will correspond to two different objects):

    colevel room=42 building=2 floor=1 wing=East room_type=Luxe sensor=A
    colevel room=42 building=2 floor=1 wing=East room_type=Luxe sensor=B

# Series names

Measurement name and the set of tags defines unique series name. Akumuli expects measurement name to be followed by the set of tags. The particular ordering of tags doesn’t matter. This is an example of the series name:

    temperature room=42 building=2 floor=1 wing=East room_type=Luxe
    humidity room=42 building=2 floor=1 wing=East room_type=Luxe
    colevel room=42 building=2 floor=1 wing=East room_type=Luxe

These are three series names. All correspond to the same object (room) but different properties. The first one tracks temperature in the room, the second one tracks humidity, and the last one is a carbon monoxide level in the room.

Another example of the series name from the DevOps monitoring world:

    mem.commit OS=Ubuntu_16.04 region=ap-southeast-1 host=PG-mirror host_IP=172.16.254.1 team=NJ instance-type=m3.2xlarge arch=x64 rack=64

This one tracks memory usage on the server. Note that the set of tags is largely redundant. It’s enough to have only “host” or “host_IP” tag to uniquely identify the series. All other tags can be used to provide rich analytics, e.g. you may want to know the average memory use by instance type.
Note that two series names correspond to the same object only if their tags are equal. This also mean that if you add one tag to series name, the name (and the series) will be different. But you can ignore some tags in queries if you want to collapse several series into one, Akumuli’s query language allows this. So, if you have several carbon monoxide sensors in the room:

    colevel room=42 building=2 floor=1 wing=East sensor=A
    colevel room=42 building=2 floor=1 wing=East sensor=B

You will be able to collapse these two series into one that contains readings from both sensors using the query language.

# Data points

Each data point should contain full series name, the timestamp, and the value. The series name, as described above, should contain metric name and the set of tags.

    <metric-name> <tag1>=<tag-value1> <tag2>=<tag-value2>...<tagN>=<tag-valueN>

The particular order of the tags doesn’t matter, both “metric tag1=1 tag2=2” and “metric tag2=2 tag1=1” correspond to the same series. The metric name, tag names, and tag values can contain any characters except the space character. The series name “mem commit OS=Ubuntu 16.04 region=ap-southeast-1 host=PG-mirror host_IP=172.16.254.1” is invalid because metric name (mem commit) and one of the tag values (Ubuntu 16.04) contains space character. Also, there shouldn’t be any spaces inside the key-value pair before or after the ‘=’ symbol. The tag “region = ap-southeast-1” is wrong, it should be “region=ap-southeast-1”.
The series shouldn’t be created beforehand. If you write a data point with new series name Akumuli will create new series automatically.

The timestamp can be a simple integer or datetime in ISO 8601 format (Akumuli only supports combined datetime representation in basic form, like in 20170405T123000.000001001).

All data points of each series should be sorted by the timestamp. You can write to the different series in any order but each series should receive datapoints with increasing timestamps. If you’ve written a data point with the timestamp set to 20170405T123000.099 to some series and now you’re writing a new data point with the timestamp field set to 20170405T122959.001 you will get a “late write” error.
The value can be an integer or a floating point number. The recommended formatting method is to use a “%.17g” format string (using printf syntax) or any equivalent. This guarantees that the precision won’t be lost.
