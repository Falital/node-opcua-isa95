[![OPC UA](http://b.repl.ca/v1/OPC--UA-ISA95-orange.png)](http://opcfoundation.org/)
[![NPM version](https://badge.fury.io/js/node-opcua-isa95.png)](http://badge.fury.io/js/node-opcua-isa95)
[![NPM download](https://img.shields.io/npm/dm/node-opcua-isa95.svg)](http://www.npm-stats.com/~packages/node-opcua-isa95)
[![Build Status](https://travis-ci.org/node-opcua/node-opcua-isa95.svg?branch=master)](https://travis-ci.org/node-opcua/node-opcua-isa95)
[![HitCount](http://hits.dwyl.io/node-opcua/node-opcua-isa95.svg)](http://hits.dwyl.io/node-opcua/node-opcua-isa95)

# ISA95 Common Object Model 

## an extension package for node-opcua

This NPM module implements the ISA95 extensions for node-opcua, to ease the
creation of ISA95 OPCUA server, in the industrial world.

It follows the specification in [OPCUA ISA95 Specification](https://opcfoundation.org/developer-tools/specifications-unified-architecture/isa-95-common-object-model)

## installation

* install node-opcua-isa95 module along side node-opcua

```
npm install node-opcua  --save
npm install node-opcua-isa95 --save
```

* use ```require("node-opcua-isa95")(opcua);``` to enrich node-opcua with the ISA95 add-on

```javascript
var opcua = require("node-opcua");

// add server ISA95 extension to node-opcua
require("node-opcua-isa95")(opcua);


var xmlFiles = [
    opcua.standard_nodeset_file,
    opcua.ISA95.nodeset_file
];

var server = new opcua.OPCUAServer({
    nodeset_filename: xmlFiles,
// details left out for clarity
});


function post_initialize() {
    console.log("initialized");
    var addressSpace = server.engine.addressSpace;

    // add your ISA95 node in the addressSpace here
    server.start(function() {
        console.log("Server is now listening ... ( press CTRL+C to stop)");
        console.log("port ", server.endpoints[0].port);
        var endpointUrl = server.endpoints[0].endpointDescriptions()[0].endpointUrl;
        console.log(" the primary server endpoint url is ", endpointUrl );
    });
}

server.initialize(post_initialize);

```

## Projects using node-opcua-isa95

ISA95 extension is ready to use with [Node-RED][1] via [node-red-contrib-iiot-opcua][2]

# API documentation

## Equipment

ISA 95 logical Equipments can be added to the addressSpace and defined using the following API.

### opcua.AddressSpace#addEquipmentClassType(options)

add a new EquipmentClassType to the addressSpace

| options                           | type                          | description                                                        |
|-----------------------------------|-------------------------------|--------------------------------------------------------------------|
| options.browseName                | {String/QualifiedName}        | the browseName of the new node equipment                           |
| [options.equipmentLevel]          | {EquipmentLevel}              | the EquipmentLevel                                                 |

#### example

```javascript
var weldingRobotClassType = addressSpace.addEquipmentClassType({
    browseName: "WeldingRobotClassType",
    equipmentLevel: opcua.ISA95.EquipmentLevel.EquipmentModule
});

```

### opcua.AddressSpace#addEquipmentType(options)

add a new EquipmentType to the addressSpace

| options                           | type                          | description                                                        |
|-----------------------------------|-------------------------------|--------------------------------------------------------------------|
| options.browseName                | {String/QualifiedName}        | the browseName of the new node equipment                           |
| [options.equipmentLevel]          | {EquipmentLevel}              | the EquipmentLevel                                                 |
| [options.definedByEquipmentClass  | {UAObjectType/ Array<UAObjectType> }  | a collection of EquipmentClassType that define this new EquipmentType |


#### Example of addEquipmentClassType

```javascript
var multiPurposeRobotType = addressSpace.addEquipmentClassType({
    browseName: "MultiPurposeRobotType",
    definedByEquipmentClass: [
        weldingRobotClassType,
        assemblingRobotClassType,
    ]
});
```

### opcua.AddressSpace#addEquipment(options)

add a new equipment to the addressSpace

| options                           | type                          | description                                                        |
|-----------------------------------|-------------------------------|--------------------------------------------------------------------|
| options.browseName                | {String/QualifiedName}        | the browseName of the new node equipment                           |
| [options.typeDefinition]          | {String/UAObjectType}         | "EquipmentType" the ISA95 type of the equipment to instantiate. if not specified, EquipmentType will be used the provided typeDefinition must be a sub type of "EquipmentType". |
| [options.definedByEquipmentClass] | {UAObjectType/Array<UAObjectType>} | The class that define the EquipmentType. It must be a subtype of "EquipmentClassType"          |
| [options.containedByEquipment]    | {UAObject}                    | the equipment that contains the created equipment. equipment must have a typeDefinition of type "EquipmentType"                         |
| [options.organizedBy]             | {UAObject}                    | a folder {FolderType} that organises the created equipment                      |


#### Example of addEquipment

```javascript
var weldingRobot = addressSpace.addEquipment({
  browseName: "WeldingRobot",
});

var robotController = addressSpace.addEquipment({
  browseName: "RobotController",
  containedByEquipment: weldingRobot
});

var robotArm = addressSpace.addEquipment({
  browseName: "RobotArm",
  containedByEquipment: weldingRobot
});
```

## PhysicalAsset

### opcua.AddressSpace#addPhysicalAssetClassType

### opcua.AddressSpace#addPhysicalAssetType

#### Example of addPhysicalAssetType

```javascript
var fanuc_robotArcMate = addressSpace.addPhysicalAssetType({
    browseName: "ArcMate 100iB/6S i",
    modelNumber: "ArcMate 100iB/6S i",
    manufacturer: {
        dataType: "String",
        value: { dataType: opcua.DataType.String, value: "Fanuc Inc"}
    }
});

addressSpace.addISA95Attribute({
    ISA95AttributeOf: fanuc_robotArcMate,
    browseName: "Weight",
    description: "Robot mass in kg",
    dataType:"Double",
    value: { dataType: opcua.DataType.Double, value: 135 },
    modellingRule: "Mandatory"
});

addressSpace.addISA95Attribute({
    ISA95AttributeOf: fanuc_robotArcMate,
    browseName: "Payload",
    description: "Payload in kg",
    dataType:"Double",
    value: { dataType: opcua.DataType.Double, value: 6 },
    modellingRule: "Mandatory"
});
addressSpace.addISA95Attribute({
    ISA95AttributeOf: fanuc_robotArcMate,
    browseName: "Repeatability",
    description: "+/-",
    dataType:"Double",
    value: { dataType: opcua.DataType.Double, value: 0.08 },
    modellingRule: "Mandatory"
});
```

### AddressSpace#addPhysicalAsset

#### Example of PhysicalAssetSet

```javascript
// create the physical asset set storage  folder
// where all our main assets will be listed
var namespace = addressSpace.getOwnNamespace();
var physicalAssetSet = namespace.addObject({
    browseName: "PhysicalAssetSet",
    typeDefinition: "FolderType",
    organizedBy: addressSpace.rootFolder.objects,
});

var robot_instance = addressSpace.addPhysicalAsset({
    organizedBy:     physicalAssetSet,
    typeDefinition:  fanuc_robotArcMate,
    definedByPhysicalAssetClass: "PhysicalAssetClassType",
    browseName: "FANUC Arc Mate 100iB/6S i - 001",
    implementationOf: robotArm,
    vendorId: {
        dataType: "String",
        value: { dataType: opcua.DataType.String, value: "RobotWox" }
    }
});
```

## Misc

### opcua.AddressSpace#addISA95Attribute

| options                           | type                          | description                                                        |
|-----------------------------------|-------------------------------|--------------------------------------------------------------------|
| options.browseName                | {String/QualifiedName}        | the browseName of the new node equipment                           |
| options.dataType                  | {String/UADataType}           | the Variable DataType                           |
| options.value                     | {Variant}                     | the Variable Value         |
| options.ISA95AttributeOf          | {UAObject/UAObjecType}        | the parent node |
| [options.modellingRule]           | {String}                      | "Mandatory" or "Optional"|
| [options.typeDefinition]           | {UAVariableType}              ||

returns a ```opcua.UAVariable````

### opcua.AddressSpace#addISA95Property

| options                           | type                          | description                                                        |
|-----------------------------------|-------------------------------|--------------------------------------------------------------------|
| options.browseName                | {String/QualifiedName}        | the browseName of the new node equipment                           |
| options.dataType                  | {String/UADataType}           | the Variable DataType                           |
| options.value                     | {Variant}                     | the Variable Value         |
| options.ISA95PropertyOf           | {UAObject/UAObjecType}        | the parent node |
| [options.modellingRule]           | {String}                      | "Mandatory" or "Optional"|
| [options.typeDefinition]           | {UAVariableType}              ||

returns a ```opcua.UAVariable````

### opcua.AddressSpace#addISA95ClassProperty

| options                           | type                          | description                                                        |
|-----------------------------------|-------------------------------|--------------------------------------------------------------------|
| options.browseName                | {String/QualifiedName}        | the browseName of the new node equipment                           |
| options.dataType                  | {String/UADataType}           | the Variable DataType                           |
| options.value                     | {Variant}                     | the Variable Value         |
| options.ISA95ClassPropertyOf      | {UAObject/UAObjecType}        | the parent node |
| [options.modellingRule]           | {String}                      | "Mandatory" or "Optionnal"|
| [options.typeDefinition]          | {UAVariableType}              |                                  |

returns a ```opcua.UAVariable````

### opcua.ISA95.EquipmentLevel

(see  Table 36 – ISA95EquipmentElementLevelEnum Values )

This DataType is an enumeration that defines the equipment element levels defined in ISA-95. Its values are
defined in a below table.

| Value       | Description |
|-------------|-------------|
| Enterprise  | An enterprise is a collection of sites and areas and represents the top level of a role based equipment hierarchy.|
| Site        | A site is a physical, geographical, or logical grouping determined by the enterprise. It may contain areas, production lines, process cells, and production units. |
| Area        | An area is a physical, geographical, or logical grouping determined by the site. It may contain work centres such as process cells, production units, production lines, and storage zones. |
| ProcessCell | Process cells are the low level of equipment typically scheduled by the Level 4 and Level 3 functions for batch manufacturing processes. |
| Unit        | Units are low level of equipment typically scheduled by the Level 4 and Level 3 functions for batch  manufacturing processes and continuous manufacturing processes. |
| ProductionLine | Production lines are low levels of equipment typically scheduled by the Level 4 or Level 3 functions for discrete manufacturing processes. |
| WorkCell       | Work cells are low levels of equipment typically scheduled by the Level 4 or Level 3 functions for  discrete manufacturing processes. |
| ProductionUnit | Production units are the lowest level of equipment typically scheduled by the Level 4 or Level functions for continuous manufacturing processes. |
| StorageZone    | Storage zones are low level of material movement equipment typically scheduled by the Level 4 and Level 3 functions for discrete, batch and continuous manufacturing processes. |
| StorageUnit    | Storage units are low level of material movement equipment typically scheduled by the  Level 4 and Level 3 functions for discrete, batch and continuous manufacturing processes.
| WorkCenter     | Work centres are typically the grouping of equipment scheduled by the Level 4 or Level 3 functions. |
| WorkUnit       | A work unit is any element of the equipment hierarchy under a work centre. Work units are the lowest  form of elements in an equipment hierarchy that are typically scheduled by Level 3 functions.|
| EquipmentModule| An equipment module entity is an engineered subdivision of a process cell, a unit, or another equipment module. |
| ControlModule  | A control module entity is an engineered subdivision of a process cell, a unit, an equipment module, or another control module. |
| Other          | non of the above |

# license
MIT

# Copyright
Copyright 2016 - Etienne Rossignon

[1]:https://github.com/node-red/node-red
[2]:https://github.com/biancode/node-red-contrib-iiot-opcua
