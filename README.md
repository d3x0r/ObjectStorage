
# Object Storage


Object storage interface is a hybrid combination of virtual file system and JSOX for object encoding.

## Dependancies

- jsox ; [NPM](https://npmjs.com/package/jsox) [Github](https://github.com/d3x0r/JSOX)
- salty random generator; [NPM](https://npmjs.com/package/@d3x0r/srg) [Github](https://github.com/d3x0r/srg)

## Server Support

- sack.vfs; [NPM](https://npmjs.com/package/sack.vfs) [Github](https://github.com/d3x0r/sack.vfs)


This is a remote storage interface


``` js

import {ObjectStorage} from "@d3x0r/object-storage"

```



| Object Storage constructor arguments | Description  |
|----|----|
| WebSocket | An optional websocket connection to an object storage server. If not provided, falls back to localStorage. |

Once you have an object storage instance, the storage is accessed primarily with `get()` and `put()` methods.

| Object methods | Return | Arguments | Description |
|----------|------------|-------------|----|
| getRoot | promise | () | Promise resolves with root directory |
| put | unique ID | ( object \[, options\] ) | Stores or updates an object into storage.  If the object was not previously stored or loaded, a unique ID for the object is returned, otherwise the existing ID results.  The second parameter is an optional option object, see options below. |
| get | promise |  ( [string] or [Option object] ) | Returns a promise; success is passed the object loaded from storage.  Loads an object using the specified ID. |
| map | promise |  ( id ) | Same as get, but also loads any objects of specified ID. |
| addDecoders |  | ( array of decoders ) | Adds additional decoder values for stored objects |
| addEncoders |  | ( array of encoders ) | Adds additional encoder values for stored objects |
| stringify | string | (object,...) | calls the internal JSOX stringifier on the object; using specified extra decoders |
| -- | | | |
| delete |  | (id/object) | remove object from storage... (danging references in other stored records?) |
| on | | ( ID, callback ) | When the specified ID changes, the callback is called with the conent of the object. |


## get() options

When getting an object, either its ID may be used directly, or the ID and optional extra decoders for the specified
object are provided.  Extra Decoders may also be assigned to the storage object.

| Object get Options names | type | Description |
|-----|-----|-----|
| id | string | use this id for the object storage target |
| extraDecoders | [ { tag:"tag", p:Type } ] | An array of optional object decoder(parse) types to use when getting objects. |


| Object put Options names | type | Description |
|-----|-----|-----|
| id | string | use this id for the object storage target |
| sign | bool | whether this record should be digitally signed; makes record readonly |
| key | string | (todo) use to make record readable only with specified key. |
| extraEncoders | [ { tag:"tag", p:Type } ] | An array of optional object encoder(stringify) types to use when putting objects. |
| fast | bool | TODO: use a memory only fast cache - for small shared data |
| local | bool | TODO: prefer local get |


These are some other options under developemnt...

| Object put Options names | type | Description |
|-----|-----|-----|
| signed | string | this data should be signed with a permanent id; record becomes readonly. |
| readKey | string | data to prevent foreign read |
| sealant | string | data to encode object id |
| objectHash | string | something |


## Simple File System Emulation

This provides a simple file system interface on top of objects.  It provides a known root identifier to
track other object indentifiers which have otherwise random values.

``` js
const storage = new ObjectStorage();


const dir = await storage.getRoot();

```
 
|Object Storage Directory Methods | Return | arguments | Description |
|----|----|----|---|
| create | FileEntry | (filename) | creates a new file, ready to be written into. |
| open | FileEntry | ( filename ) | opens a file relative to the current root. |
| folder | FileDirectory | ( pathname ) | gets a path within the current path. |
| store | none | () | saves any changes. |
| has | Boolean | ( pathanem ) | returns true/false indicating whether this directory contains the specified pathname. |
| remove | Promise | ( filename ) | delete specified file from the file directory. |


Once you have a directory you can load file content from it.

```
const file = await dir.open( "filename" );
file.read();
file.write( content );
```

|Object Storage File Methods | Return | arguments | Description |
|----|----|----|---|
| open | FileDirectory | ( pathane ) | opens a file within this file as a folder |
| getLength | Number | () | returns the current length of this file |
| read | Promise | ( [pos,] length) | Takes optional parameters (length), or (position, length); returns a promise that resolves with an ArrayBuffer |
| write | Promise | ( ArrayBuffer | String ) | Write this array buffer or string to the file.  Promise resolves with the id of the object written. |


``` js
storage.getRoot().then( 
	root=>root.open("filename")
        	.catch( ()=>root.create( "filename" ) )
                	.then( (file)=>file.read().then( data=>{
			   console.log( "File Data:", data );
		} ) ) )

storage.getRoot().then( root=>
	root.open("filename")
		.catch( ()=>
			root.create( "filename" ).then( file=>{
				file.write( "Initial content?" ); 
			} )
		.then( (file)=>
			file.read()
				.then( data=>{
					console.log( "File Data:", data );
				} ) 
			) 
		)

```


