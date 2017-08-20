# SimpleXdmf
A C++ header-only library to produce XDMF format file.
SimpleXdmf does not have complete functionalities for Xdmf, but does not require building with HDF5 (and Boost).

# Installation
Clone it into your project.
```bash
git clone https://github.com/hsimyu/SimpleXdmf.git
```
and copy a header file "simple_xdmf.hpp" to your include directory.

or, if your project already used cmake to build,
add the line below into your CMakeLists.txt.
```cmake
add_subdirectory(SimpleXdmf)
```
Turn ON options for cmake if you want include SimpleXdmf tests and examples building.

# How To Use
```cpp
#include <simple_xdmf.hpp>

int main() {
    SimpleXdmf gen;

    // Some settings
    gen.setNewLineCodeLF();
    gen.setIndentSpaceSize(4);

    const int nx = 5;
    const int ny = 3;

    gen.beginDomain("Domain1");
        gen.beginGrid("Grid1");
            gen.beginStructuredTopology("Topo1", "2DCoRectMesh");
            gen.setNumberOfElements(ny, nx);
            gen.endStructuredTopology();

            gen.beginGeometory("Geom1", "ORIGIN_DXDY");
            gen.setDimensions(ny, nx);
                // Origin
                gen.beginDataItem();
                    gen.setDimensions(2);
                    gen.setFormat("XML")
                    gen.addItem(0.0, 0.0);
                gen.endDataItem();

                gen.beginDataItem();
                    gen.setDimensions(2);
                    gen.setFormat("XML");
                    gen.addItem(0.5, 0.5);
                gen.endDataItem();
            gen.endGeometory();

            gen.beginAttribute("Attr1");
            gen.setCenter("Node");
                gen.beginDataItem();
                    gen.setDimensions(ny, nx);
                    gen.setFormat("XML");

                    // Adding from std::vector
                    std::vector<float> node_values;

                    for(int i = 0; i < (nx * ny); ++i) {
                        node_values.emplace_back(i);
                    }

                    gen.addVector(node_values);
                gen.endDataItem();
            gen.endAttribute();
        gen.endGrid();
    gen.endDomain();

    gen.generate("test.xmf");

    return 0;
}
```

This sample code generates
```xml
<?xml version="1.0" ?>
<!DOCTYPE Xdmf SYSTEM "Xdmf.dtd" []>
<Xdmf>
    <Domain Name="Domain1">
        <Grid GridType="Uniform">
            <Topology TopologyType="2DCoRectMesh" NumberOfElements="3 5" Name="Topo1"/>
            <Geometry GeometryType="ORIGIN_DXDY" Dimensions="3 5" Name="Geom1">
                <DataItem DataItemType="Uniform" Dimensions="2" Format="XML">
                    0 0 
                </DataItem>
                <DataItem DataItemType="Uniform" Dimensions="2" Format="XML">
                    0.5 0.5 
                </DataItem>
            </Geometry>
            <Attribute AttributeType="Scalar" Center="Node" Name="Attr1">
                <DataItem DataItemType="Uniform" Dimensions="3 5" Format="XML">
                    0 1 2 3 4 5 6 7 8 9 
                    10 11 12 13 14
                </DataItem>
            </Attribute>
        </Grid>
    </Domain>
</Xdmf>
```

## Correct order of calling functions
SimpleXdmf assumes begin/add/end structure.
When you insert a new tag, first please call a begin*() function.
After you called begin, set*() functions can set a passed argument for the current tag.
After you inserted arbitrary elements within the tag, please call the corresponding end*() function.

add*() functions insert raw elements without tag.
Use it to describe values in DataItem tag.

## Supported Functions
Supported begin/end functions are
- beginDomain(const std::string& name = "") / endDomain();
- beginGrid(const std::string& name = "", const std::string& GridType = "Uniform") / endGrid();
- beginStructuredTopology(const std::string& name = "", const std::string& TopologyType = "2DCoRectMesh") / endStructuredTopology();
- beginUnstructuredTopology(const std::string& name = "", const std::string& TopologyType = "Polyvertex") / endUnstructuredTopology();
- beginGeometry(const std::string& name = "", const std::string& GeometryType = "XYZ") / endGeometry();
- beginDataItem(const std::string& name = "", const std::string& DataItemType = "Uniform") / endDataItem();
- beginAttribute(const std::string& name = "", const std::string& AttributeType = "Scalar") / endAttribute();
- beginSet(const std::string& name = "", const std::string& SetType = "Node") / endSet();
- beginTime(const std::string& name = "", const std::string& TimeType = "Single") / endTime();
- beginInformation(const std::string& name = "") / endInformation();

All method can be passed its Name attribute as the 1st argument, and its type as the 2nd argument when non-default type is required.

Supported set functions are
- setName(const std::string& Name)
- setVersion(const std::string& Version)
- setFormat(const std::string& Format)
- setPrecision(const std::string& Precision)
- setNumberType(const std::string& NumberType)
- setCenter(const std::string& Center)
- setFunction(const std::string& Function)
- setSection(const std::string& Section)
- setValue(const std::string& Value)
- setReference(const std::string& Reference)
- setReferenceFromName(const std::string& Name) (see below)
- setDimensions(Args&&... args)
- setNumberOfElements(Args&&... args)

Supported add functions are
- addItem(Args&&... args)
- addVector(const std::vector<T>& values)
- addReferenceFromName(cosnt std::string& Name) (see below)

Some configure functions are defined.
- setNewLineCodeLF()
- setNewLineCodeCR()
- setNewLineCodeCRLF()
- setIndentSpaceSize(const int size = 4); if size = 0, use '\t'.

I/O function is now only one.
- void generate(const std::string& filename)
- std::string getRawString()

## Reference management
SimpleXdmf also have a simple reference management.
setReferenceFromName() and addReferenceFromName() functions automatically set the Xpath if the passed name exists.

```cpp
#include <simple_xdmf.hpp>

int main() {
    SimpleXdmf gen;

    const int nx = 5;
    const int ny = 3;

    gen.beginDomain();
        gen.beginGrid();
        gen.setName("Grid1");
        gen.endGrid();

        gen.beginGrid();
        gen.setName("Grid2");
        gen.setReferenceFromName("Grid1"); // set Xpath as Reference attribute
        gen.endGrid();

        gen.beginGrid();
        gen.setName("Grid3");
            gen.addReferenceFromName("Grid3"); // as inner reference element
        gen.endGrid();

        gen.beginGrid();
        gen.setName("Grid4");
        gen.setReference("/Xdmf/Domain/Grid2"); // or directly setting
        gen.endGrid();
    gen.endDomain();

    gen.generate("reference_management.xmf");
    return 0;
}
```

This sample code generates

```xml
<?xml version="1.0" ?>
<!DOCTYPE Xdmf SYSTEM "Xdmf.dtd" []>
<Xdmf>
    <Domain>
        <Grid GridType="Uniform" Name="Grid1">
        </Grid>
        <Grid GridType="Uniform" Name="Grid2" Reference="/Xdmf/Domain/Grid[@Name='Grid1']">
        </Grid>
        <Grid GridType="Uniform" Name="Grid3">
            /Xdmf/Domain/Grid[@Name='Grid3'] 
        </Grid>
        <Grid GridType="Uniform" Name="Grid4" Reference="/Xdmf/Domain/Grid2">
        </Grid>
    </Domain>
</Xdmf>
```

For details, see the examples in the examples directory.

# License
MIT
