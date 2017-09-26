# gs-JSON

gs-JSON is a general file format for modelling with geometry and semantics. The geometry includes both polygonal and spline based geometry. Semantics can be added to any geometric entity or collection of entities.

# Geometry

Geometric entities are interconnected into a topological hierarchy. 

## Topological Hierarchy

The topological hierarchy is follows:
* 0D
  * VERTEX = geometry
* 1D
  * EDGE = geometry bounded by VERTICES
  * WIRE = geometry: a set of one or more VERTEX connected EDGES
* 2D
  * FACE = geometry bounded by one or more closed WIRES

Each WIRE has:
* a set of connected EDGES (implicit), each of which has
* a sequence of VERTICES (implicit), each of which is
* associated with a single point.
    
Each FACE has:
* a set of closed WIRES (implicit), each of which has
* a set of connected EDGES (implicit), each of which has
* a sequence of VERTICES (implicit), each of which is
* associated with a single point.
    
Certain levles of geometric entities are not explicitly represented. The exist implicitly. For example, a polygonal or polyline has implicit EDGES and VERTICES.

Other higher level topologies (such as shells, solids, and compound solids) can be created using *collections*. See below for more details. 

## Geometric Entities

All geometric entities references a list of POINTS.

The geometric entities are as follows:
* 0D VERTICES:
  * 0 - Acorn
  * 1 - Ray
  * 2 - Plane
* 1D WIRES, an array of:
  * 100 - Polyline
  * 130 - NURBS curve
  * 131 - Bezier curve
* 2D FACES:
  * 200 - Polygon
  * 230 - NURBS Surface
  * 231 - Bezier Surface

Each geometric entity is defined as by an array of three elements as follows: 
* [type, [vertex indices], additional parameters]

So, for example, a polyline is defined as follows:
* [100,[[0,0],[0,1],[0,2],[0,3]],0]

This represents the following:
1. type = 100, i.e. polyline
1. vertex indices = [[0,0],[0,1],[0,2],[0,3]]
1. additional parameters = 0, open polyline
    
The third element may in some cases be omitted. 
For example, a polygon has no parameters. 

# Semantics

Semantic information can be added to the model in two ways: *attributes* and *collections*. 

## Attributes

Attributes can be defined for POINTS, VERTICES, EDGES, WIRES and FACES. When an attribute is defined, all entities of that type will be assigned a value.

The lists of values may typically be sparse (i.e. there may be many 'null' values) and may contain many repeat values. An efficient reveresed key-value datastructure is used. (The keys are the attribute values, and the values are arrays of entity numbers.)

For example, for the list of values below:
* [null,'a','b','c','a',null,null,null,'b','c','b','c','a','b','b','c',null,null]
* 0     1   2   3   4  5    6    7     8   9   10  11  12  13  14  15 16   17 <- indexes

The represented is as follows:
    {
        "a":[1,4,12],
        "b":[2,8,10,13,14],
        "c":[3,9,11,15]
    }

## Collections


Collections can have user defined properties, which are define as key-value pairs.
Collections are reference by their unique name.

A collection can contain:
* a set of entities (possibly mixed topologival types, including implicit entities), and
* other collections.

The set of entities inside a collection are defined by indexing the geometry (i.e. the FACES, WIRES, EDGES, VERTICES, and POINTS). 

## Indexing Geometry

In order to identify the entities and sub-entities in a collection, indexing arrays are used.

The basic form of these arrays is as follows:
* For WIRES: [wire_number, edge_number, vertex_number]
* For FACES: [face_number, wire_number, edge_number, vertex_number]
For example:
* wire 0, edge 1, vertex 0:
  * [0,1,0] 
* face 0, wire 1, edge 2, vertex 0:
  * [0,1,2,0] 

An index array may be truncated.
For example, 
* face 0, wire 1, edge 2:
  * [0,1,2] No need to specify any vertices.

An index value may use right side indexing, i.e. negative numbers (c.f. Python slicing).
For example:
* face 0, wire 1, last edge:
  * [0,1,-1]

An index value may be 'null', indicating that the level should be skipped.
For example:
* face 0, skip wires, skip edges, vertex 10:
  * [0,null,null,10]

An index value may specofy a range, as follows: [from, to, step]. The 'step' may be omitted, in which case it is assumed to be 1. 
For example:
* face 0, wire 1, edges 2 to 4:
  * [0,1,[2,4]]
* face 0, wire 1, last three edges:
  * [0,1,[-3,-1]]
* face 0, wire 1, all edges:
  * [0,1,[0,-1]]

The above may all be combined.
For example:
* face 0, skip wires, every other edge, start vertex:
  * [0,null,[0,-1,2],0]

# Example

WORK IN PROGRESS.

Note: javascript comments are used in these examples even though comments are not technically allowed in JSON.

```javascript
{
    //---------------------------------------------------------------------------------------------
    //Some meta data
	"metadata": {
		"filetype":"mobius",
        "version": 1.0,
        "schema":"xxx",
        "crs": {"epsg":3857},
		"location": "+40.6894-074.0447" //ISO 6709, ±DD.DDDD±DDD.DDDD degrees format
	},
    //---------------------------------------------------------------------------------------------
    "skins": {
        //See https://github.com/mrdoob/three.js/wiki/JSON-Texture-format-4
        "images": [],    //based on three.js
        "textures": [],  //based on three.js
        "materials": [  //based on three.js
            {...},
            {...},
            {...}
        ], 
    }
    //---------------------------------------------------------------------------------------------
    "geometry": {
        "points": [
            [[1.2,3.4],[5.6,7.8],[9.10,11.12], ....],           //list of 2d [x,y] coordinates
            [[0.1,0.2,0.3],[1.4,1.5,1.6],[2.7,2.8,2.9], ....],  //list of 3d [x,y,z] coordinates
            [[1.1,1.2,1.3],[2.4,2.5,2.6],[3.7,3.8,3.9], ....]   //list of 3d [x,y,z] coordinates
        ]
        "xforms": [
            [1,0,0,20, 0,1,0,0, 0,0,1,0, 0,0,0,1],   //point transformation matrix, 4x4
            [1,0,0,30, 0,1,0,0, 0,0,1,0, 0,0,0,1],   //point transformation matrix, 4x4
            [1,0,0,40, 0,1,0,0, 0,0,1,0, 0,0,0,1]    //point transformation matrix, 4x4
        ],
        "vertices": [
            [0, [0,0]],            //acorn   [type, [origin vtx_id]]
            [1, [0,0], [1,1,1]],   //ray     [type, [origin vtx_id], [ray vector]]
            [2, [1,1], [1,0,0]]    //plane   [type, [origin vtx_id], [plane normal vector]]
        ]
        "wires": [
            [100, [[0,0],[0,1],[0,2],[0,3]], 0],   //planar open polyline, open (3 edges)  [type, [vtx_ids], [open_closed]]
            [100, [[1,0],[1,1],[1,2],[1,3]], 1],   //3d closed polylines (4 edges)         [type, [vtx_ids], [open_closed]]
        ]
        "faces": [
            [200, [[[2,50],[2,51],[2,52],[2,53]]]],                //polygon              [type, [[periphery vtx_ids]], []]
            [200, [[[1,60],[1,61],[1,62]],[[1,3],[1,4],[1,5]]]],   //polygon with a hole  [type, [[periphery vtx_ids],[hole 1 vtx_ids],[hole 1 vtx_ids]]]
        ]
    }
    //---------------------------------------------------------------------------------------------
    "semantics": {
        "attributes": [
            {//some data attached to all the POINTS 
                "uuid":"xxxxx",
                "name":"trees",
                "level":"points", 
                "values": {
                    "raintree.czml":[1,3,5,6,7,8,11,...],
                    "oaktree.czml":[2,3,20,22,...],
                    ...
                }
            },
            {//some data attached to all the implicit EDGES
                "uuid":"xxxxx",
                "name":"construction",
                "level":"edges" 
                "values": {
                    "timber":[5,23,67,99,...],
                    "steel":[25,27,44,52,...],
                    "concrete":[1,45,46,87,...],
                    ...
                }
            },
            {//some data attached to all the FACES
                "uuid":"xxxxx",
                "name":"insolation",
                "level":"faces" 
                "values": {
                    123:[1],
                    567:[2],
                    264:[3],
                    422:[4],
                    124:[5],
                    ...
                }
            },
            {//the viewer may "recognise" this attrib and render the geometry accordingly
                "uuid":"xxxxx",
                "name":"materials",
                "level":"faces"
                "values": {
                    0:[1,2,4,6,7],
                    1:[8,9,12,44,66],
                    2:[55,77],
                    ...
                }            
            },
            {//the viewer may "recognise" this attrib and render the geometry accordingly
                "uuid":"xxxxx",
                "name":"color",
                "level":"vertices"
                "values": {
                    [0.3,0.2,0.4]:[1,2,4,6,7],
                    [0.7,0.2,0.3]:[8,9,12,44,66],
                    ...
                }            
            },
            {//the viewer may "recognise" this attrib and render the geometry accordingly
                "uuid":"xxxxx",
                "name":"normals":
                "level":"vertices"
                "values": {
                    [0.0,0.0,1.0]:[1,3,5,7,9,...],
                    [0.0,1.0,1.0]:[2,4,6,8,...],
                    [1.0,0.0,1.0]:[10,20,30,40,...],
                    ....
                }
            }
        },
        "collections": {
            {//Empty collection (which is ok), it has some properties
                "uuid":"xxxxx", 
                "name":"no_geometry",
                "properties": {"key1":value1, "key2":value2, ...},
            },
            {//A collection containing two EDGES. It has no properties (which is ok).
                "uuid":"xxxxx",  
                "name":"some_edges", //user defined name
                "faces":[[0,0,[0,-1,2]]], //first face, first wire, every other edge (uses ranges)
            },
            {//A collection containing two WIRES
                "uuid":"xxxxx", 
                "name":"two_wires",
                "faces":[[1,0],[1,1]],
                "properties": {"key1":value1, "key2":value2, ...}
            },
            {//A collection containing some other collections.
                "uuid":"xxxxx", 
                "name":"coll_of_colls",
                "collections":["no_geometry", "one_vertex"],
                "properties": {"key1":value1, "key2":value2, ...}
            },
            {//A collection containing some random stuff.
                "uuid":"xxxxx", 
                "name":"everything_all_mixed_up",
                "vertices":[0,1,2],                         //three vertices
                "wires":[[0,1],[1,0]],                      //two edges, in wires
                "faces":[[0,0,1],[1,1,0]],                  //another two edges, in faces
                "collections":["coll_of_colls"],            //a collection
                "properties": {"key1":value1, "key2":value2, ...}
            }
        }
    }
    //---------------------------------------------------------------------------------------------
}
````
