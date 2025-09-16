# Node Code Reference for Developers (Low Code Platform) – iConnector

## 1. Purpose of this Document

* Enable both developers and AI Assistants to write, review, and recommend Node code correctly, safely, and in a maintainable way.
* Serve as a **self-contained** central guide that does not require referencing other documents.
* **The system that iConnector integrates with is IBM i (AS/400).**
* **Screen coordinates use row and column, starting at row:1, column:1 (top-left corner of the screen).**
* **If screen size is not specified, use the default 24 rows × 80 columns (code page: cp838).**

---

## 2. Constraints & Practices (MUST READ)

* **Do not import any additional libraries or classes** (everything must come from the system).
* Do not create new classes/methods yourself (use only what the system provides).
* Only use objects/functions injected by the system, e.g., `input`, `logger`, `UTIL`, `UTILExtension`, `EWIContextData`, `EWIContextCollection`.
* Avoid modifying the main flow structure or the system-defined context data.
* Always check for null/empty before using data (e.g., `contextData == null`, `StringUtil.isBlank(value)`).
* Use the logger for debug and trace at every important point.
* All allowed functions/objects are injected and ready to use in Node code.
* Avoid static variables, global state, or logic that causes side effects for other flows.
* **Node code must use syntax supported by JDK 1.8 only** (e.g., avoid Java 9+ syntax such as `var`, some Stream API features, overly complex lambda expressions, etc.).

---

## 3. Allowed Helpers/Functions (with examples and explanations)

### input (read context data)

* `input.getContextData(fieldName)`: fetch context data by field name (returns `EWIContextData`).
* `input.getContextData(fieldName).getValue()`: get the actual value from a field (String, Number, etc.).
* `input.getContextData(fieldName).getChildren()`: get a DataList (`List<EWIContextCollection>`).

Example:

```java
EWIContextData data = input.getContextData("NBROL");
if (data != null && !StringUtil.isBlank(data.getValue())) {
    String value = data.getValue();
}
```

### logger (log information)

* `logger.info(msg)`: general log.
* `logger.warn(msg)`: warning.
* `logger.error(msg)`: error.

Example:

```java
logger.info("Start Node: Create Attribute List");
logger.warn("Field X is null");
logger.error("Unexpected error: " + e.getMessage());
```

### UTIL (utility functions)

* `UTIL.addMapToDataList(listName, mapObj)`: append a map to a DataList.
* `UTIL.getMapValue(fieldName)`: read a Map (JSON) from a field (returns `Map<String, Object>`).
* `UTIL.setMapValue(fieldName, mapObj)`: write a Map back to a field.
* `UTIL.changeTimeFormatInList(listName, fieldName, fromFormat, toFormat, isAllRow)`: change date format in a DataList.
* `UTIL.getUUID()`: generate a 32-char UUID (String).
* `UTIL.getUUID8bit()`: generate an 8-char UUID (String).
* `UTIL.getDataListInDataList(mainList, subListField, index)`: get a nested DataList (`List<EWIContextCollection>`).

Example:

```java
Map<String, Object> map = UTIL.getMapValue("myMapField");
if (map != null) {
    Object v = map.get("subKey");
}
UTIL.setMapValue("myMapField", map);
String uuid = UTIL.getUUID();
```

### UTILExtension (additional utilities)

* Data transformation, validation, string/number handling (see method names in `helper/UTILExtension.java`).

Example:

```java
String result = UTILExtension.someUtilityMethod(args);
```

### EWIContextData, EWIContextCollection (read/write context data)

* Used to create/modify fields or rows in a DataList.

Example: create a new field in a row

```java
EWIContextData statusField = new EWIContextData();
statusField.setName("status");
statusField.setValue("fail");
row.addContextData("status", statusField);
```

Read a value from a row:

```java
String value = row.getContextData("fieldName").getValue();
```

---

## 3.1 Special Key Input

In iConnector, you can perform “Key In” or press special keys (e.g., Enter, F3, PageDown, PF12, etc.) using the provided `execute()` or `execute(null, keyString)` methods.

### How to Use

* `execute();`
  Press Enter (confirm data on the current screen).

* `execute(null, HostConstants.KEY_ENTER_STR);`
  Press Enter (same as `execute();`).

* `execute(null, HostConstants.KEY_F1_STR);`
  Press F1

* `execute(null, HostConstants.KEY_F2_STR);`
  Press F2

* `execute(null, HostConstants.KEY_F3_STR);`
  Press F3

* `execute(null, HostConstants.KEY_F4_STR);`
  Press F4

* `execute(null, HostConstants.KEY_F5_STR);`
  Press F5

* `execute(null, HostConstants.KEY_F6_STR);`
  Press F6

* `execute(null, HostConstants.KEY_F7_STR);`
  Press F7

* `execute(null, HostConstants.KEY_F8_STR);`
  Press F8

* `execute(null, HostConstants.KEY_F9_STR);`
  Press F9

* `execute(null, HostConstants.KEY_F10_STR);`
  Press F10

* `execute(null, HostConstants.KEY_F11_STR);`
  Press F11

* `execute(null, HostConstants.KEY_F12_STR);`
  Press F12

* `execute(null, HostConstants.KEY_PF1_STR);`
  Press PF1

* `execute(null, HostConstants.KEY_PF2_STR);`
  Press PF2

* `execute(null, HostConstants.KEY_PF3_STR);`
  Press PF3

* `execute(null, HostConstants.KEY_PF4_STR);`
  Press PF4

* `execute(null, HostConstants.KEY_PF5_STR);`
  Press PF5

* `execute(null, HostConstants.KEY_PF6_STR);`
  Press PF6

* `execute(null, HostConstants.KEY_PF7_STR);`
  Press PF7

* `execute(null, HostConstants.KEY_PF8_STR);`
  Press PF8

* `execute(null, HostConstants.KEY_PF9_STR);`
  Press PF9

* `execute(null, HostConstants.KEY_PF10_STR);`
  Press PF10

* `execute(null, HostConstants.KEY_PF11_STR);`
  Press PF11

* `execute(null, HostConstants.KEY_PF12_STR);`
  Press PF12

* `execute(null, HostConstants.KEY_PF13_STR);`
  Press PF13

* `execute(null, HostConstants.KEY_PF14_STR);`
  Press PF14

* `execute(null, HostConstants.KEY_PF15_STR);`
  Press PF15

* `execute(null, HostConstants.KEY_PF16_STR);`
  Press PF16

* `execute(null, HostConstants.KEY_PF17_STR);`
  Press PF17

* `execute(null, HostConstants.KEY_PF18_STR);`
  Press PF18

* `execute(null, HostConstants.KEY_PF19_STR);`
  Press PF19

* `execute(null, HostConstants.KEY_PF20_STR);`
  Press PF20

* `execute(null, HostConstants.KEY_PF21_STR);`
  Press PF21

* `execute(null, HostConstants.KEY_PF22_STR);`
  Press PF22

* `execute(null, HostConstants.KEY_PF23_STR);`
  Press PF23

* `execute(null, HostConstants.KEY_PF24_STR);`
  Press PF24

* `execute(null, HostConstants.KEY_PAGEDOWN_STR);`
  Press PageDown

* `execute(null, HostConstants.KEY_PAGEUP_STR);`
  Press PageUp

* `execute(null, HostConstants.KEY_HOME_STR);`
  Press Home

* `execute(null, HostConstants.KEY_END_STR);`
  Press End

* `execute(null, HostConstants.KEY_INSERT_STR);`
  Press Insert

* `execute(null, HostConstants.KEY_DELETE_STR);`
  Press Delete

* `execute(null, HostConstants.KEY_TAB_STR);`
  Press Tab

* `execute(null, HostConstants.KEY_BACKSPACE_STR);`
  Press Backspace

* `execute(null, HostConstants.KEY_ESC_STR);`
  Press Escape (ESC)

* `execute(null, HostConstants.KEY_UP_STR);`
  Press Up Arrow

* `execute(null, HostConstants.KEY_DOWN_STR);`
  Press Down Arrow

* `execute(null, HostConstants.KEY_LEFT_STR);`
  Press Left Arrow

* `execute(null, HostConstants.KEY_RIGHT_STR);`
  Press Right Arrow

> **Notes:**
>
> * Special key values are in the `HostConstants` class.
> * You can see all key names in HostConstants (e.g., `KEY_F3_STR`, `KEY_PF3_STR`, `KEY_PAGEDOWN_STR`, `KEY_PF12_STR`, etc.).
> * Some hosts/screens may only support certain keys.

### Examples

```java
// Press Enter (normal)
execute();
execute(null, HostConstants.KEY_ENTER_STR);

// Press F3
execute(null, HostConstants.KEY_F3_STR);

// Press PF12
execute(null, HostConstants.KEY_PF12_STR);

// Press PageDown
execute(null, HostConstants.KEY_PAGEDOWN_STR);

// Press Up Arrow
execute(null, HostConstants.KEY_UP_STR);

// Press ESC
execute(null, HostConstants.KEY_ESC_STR);
```

### Example Usage in a Flow

```java
if (checkScreen(OIS100_A)) {
    // Fill in various fields
    getService().setText(4, 18, "3");
    setContextData(input.getContextData("coNo"));

    // Press Enter to confirm
    execute();

    // Press F3 to go back
    execute(null, HostConstants.KEY_F3_STR);

    // Press PageDown to scroll
    execute(null, HostConstants.KEY_PAGEDOWN_STR);
}
```

> **Cautions:**
>
> * Always check the screen (`checkScreen`) before each `execute`.
> * Repeatedly executing without changing data may break the flow.
> * Use `logger.info` every time you call `execute` for debugging.

---

## 4. Important Use Cases (Best Practices with Code)

### 4.1 Convert fields to an array (Attribute List)

```java
String[] keys = {"NBROL", "WIDTH", "LNGTH", ...};
EWIContextData atl = input.getContextData("attributeList");
atl.getChildren().clear();
for (String key : keys) {
    EWIContextData contextData = input.getContextData(key);
    boolean isBlankValue = contextData == null || StringUtil.isBlank(contextData.getValue());
    if (contextData != null && contextData.getValue() != null && !isBlankValue) {
        Map<String, Object> item = new HashMap<String, Object>();
        item.put("attribute", key);
        item.put("attributeValue", contextData.getValue());
        UTIL.addMapToDataList("attributeList", item);
    } else {
        logger.warn("Context data for key '" + key + "' is null.");
    }
}
```

### 4.2 Read/Write a Map

```java
Map<String, Object> map = UTIL.getMapValue("myMapField");
if (map != null) {
    Object value = map.get("subKey");
    logger.info("Value of subKey: " + value);
}
Map<String, Object> newMap = new HashMap<String, Object>();
newMap.put("subKey", "newValue");
UTIL.setMapValue("myMapField", newMap);
```

### 4.3 Change date format in a DataList

```java
UTIL.changeTimeFormatInList("orderList", "orderDate", "yyyyMMdd", "dd/MM/yyyy", false);
```

### 4.4 Generate UUID

```java
String uuid = UTIL.getUUID();
String uuid8 = UTIL.getUUID8bit();
```

### 4.5 Get a nested DataList

```java
List<EWIContextCollection> subList = UTIL.getDataListInDataList("mainList", "subListField", 0);
logger.info("SubList size: " + (subList != null ? subList.size() : 0));
```

### 4.6 Add a column to a DataList

> **Cautions:**
>
> * Before iterating the `children` of a DataList, check the type with `instanceof` to ensure it’s an `EWIContextCollection`.
> * When adding new context data in a row, use methods supported by the class (e.g., `addContextData` or `setContextData`).
> * This example accounts for cases where not all `children` are `EWIContextCollection`.

```java
EWIContextData productList = input.getContextData("productList");
if (productList != null && productList.getChildren() != null) {
    for (Object obj : productList.getChildren()) {
        if (!(obj instanceof EWIContextCollection)) continue;
        EWIContextCollection row = (EWIContextCollection) obj;

        // Example: add a new field named "status"
        EWIContextData statusField = row.getContextData("status");
        if (statusField == null) {
            statusField = new EWIContextData();
            statusField.setName("status");
            row.addContextData("status", statusField); // or use setContextData if available
        }
        statusField.setValue("fail");
    }
} else {
    logger.warn("productList is null or has no children.");
}
```

---

### Additional Notes on Iterating & Updating a DataList

* If a DataList’s children are not all `EWIContextCollection`, check the type before casting.
* When adding new context data to a row, use the methods the class supports (`addContextData`, `setContextData`, or `putContextData`).
* Example of safely checking and updating a field in a row:

```java
EWIContextData productList = input.getContextData("productList");
if (productList != null && productList.getChildren() != null) {
    for (Object obj : productList.getChildren()) {
        if (!(obj instanceof EWIContextCollection)) continue;
        EWIContextCollection row = (EWIContextCollection) obj;

        EWIContextData priceField = row.getContextData("price");
        EWIContextData discountField = row.getContextData("total_discount");

        String flagValue = "N"; // default

        try {
            double price = (priceField != null && !StringUtil.isBlank(priceField.getValue()))
                           ? Double.parseDouble(priceField.getValue())
                           : 0;

            if (price > 100) {
                flagValue = "Y";
            }

            if (price < 100 && discountField != null && !StringUtil.isBlank(discountField.getValue())) {
                double originalDiscount = Double.parseDouble(discountField.getValue());
                double newDiscount = originalDiscount * 0.5;
                discountField.setValue(String.valueOf(newDiscount));
            }

        } catch (NumberFormatException e) {
            logger.warn("Invalid number in price or discount: price=" +
                        (priceField != null ? priceField.getValue() : "null") +
                        ", total_discount=" + 
                        (discountField != null ? discountField.getValue() : "null"));
        }

        EWIContextData flagField = row.getContextData("flag");
        if (flagField == null) {
            flagField = new EWIContextData();
            flagField.setName("flag");
            row.addContextData("flag", flagField); // or use setContextData if available
        }
        flagField.setValue(flagValue);
    }
} else {
    logger.warn("productList is null or has no children.");
}
```

**Summary:**

* Check the type of `children` before casting.
* Use class-supported methods to add context data.
* Always null/empty-check before using data.
* Use the logger for debugging at all important points.

---

## 5. How Node Code (generatecode) Executes

* Node code is generated and executed in the flow defined in `generatecode`.
* All allowed functions/objects are injected and ready to use.
* Avoid modifying the core context data the system’s flow relies on.
* Use `logger` for debugging/tracing.
* Code must be stateless (no state carried across executions).

---

### Summary of the DataBus (EWIContext) and the Relationship Between input / output

* **DataBus** is the main object that stores and passes data between Node code steps in an iConnector flow.
* DataBus has a structure similar to `Map<String, EWIContextData>`. Each field (or DataList) is stored as `EWIContextData`.
* Data in the DataBus can be a single field (String, Number, etc.) or a DataList (`List<EWIContextCollection>`).
* Every Node code receives **input** (`EWIContext`) and sends results via **output** (`EWIContext`), which is often the same object (e.g., `dataBus`).
* Modifying the input (e.g., adding/updating a field or DataList) affects the output immediately if input and output reference the same object (pass-by-reference).
* If you want output to be isolated from input, clone/copy the context. (In `generatecode`, a single object is usually used.)

#### input vs. output

* **input**: used to read context data passed into the Node code.
* **output**: used to write or update context data to be passed to the next Node or returned as the flow’s final result.
* In typical flows, `input` and `output` point to the same DataBus object (e.g., `processFlow(dataBus, dataBus)`).
* Therefore, changes via `input` or `output` affect the same DataBus (changes are passed along the flow).
* For example, adding a column to a DataList via `input` makes it immediately visible to `output`.

**Cautions:**

* Avoid modifying the system-critical context data unless necessary.
* Always null/empty-check before using data.
* If you need strict separation of input/output, create a new `EWIContext` and copy data (but `generatecode` typically does not use this pattern).

**Summary:**

* The DataBus (`EWIContext`) is the data hub for Node code.
* `input`/`output` are typically the same object.
* Changing `input` affects `output` immediately (pass-by-reference).
* Used to pass data continuously between Nodes/Flows.

---

## 6. Debugging & Quality Checks

* Use `logger.info/warn/error` at all key points (input, output, errors, decisions).
* Validate every field before use (null/empty).
* Test the provided use-cases before deploying.
* When errors occur, log details and relevant context data.
* Do not swallow errors (log every error that occurs).

---

## 7. Checklist for Developers/AI Assistants

* [ ] Avoid importing or creating new classes/methods.
* [ ] Use only system-provided Helpers/Functions.
* [ ] Check for null/empty every time.
* [ ] Use the logger for debugging.
* [ ] Test the example use-cases before deploying.
* [ ] Read/reference examples from `HowToWriteNodeCode.md` and files under `generatecode/`.
* [ ] If unsure, ask the development team.
* [ ] Code must be stateless and free of side effects on other flows.

---

## 8. References & Additional Resources

* Examples and best practices: `HowToWriteNodeCode.md`
* Helper/Function details: the `helper/` folder
* Real flow examples: the `generatecode/` folder
* Contact the development team: \[contact channel]

> **Note:** This document is designed so an AI or Assistant can immediately use it as a central guide to recommend/review/fix Node code without needing other docs. For more details on Helpers/Functions or specific use-cases, see `HowToWriteNodeCode.md` and code in `generatecode/`, or contact the development team.

---

## 9. Structure and Deep Details of EWIContext, UTIL, EWIContextData, EWIContextCollection

### EWIContext (input/output)

* The main object holding Node code context for both input and output.
* Structure similar to `Map<String, EWIContextData>`.
* Used to read/write fields, DataLists, and other meta information.
* Key methods:

  * `getContextData(String fieldName)`: get `EWIContextData` by field name.
  * `setContextData(EWIContextData data)`: insert/update a field.
  * `removeContextData(String fieldName)`: remove a field.
  * `getAll()`: return all `Map<String, EWIContextData>`.
  * `putAll(Map)`: insert multiple fields at once.
  * `getContextData(String[] fieldNames)`: return a `Map` of selected fields.
  * `getContextMessages()`: messages/alerts in the context.
  * `setResponseCode(String)` / `getResponseCode()`: manage response codes.
  * `setResponseMessage(String)` / `getResponseMessage()`: manage response messages.

#### Notes on input/output

* In typical `generatecode` usage, `input` and `output` are the same object (pass-by-reference).
* Changes via either `input` or `output` immediately affect the same DataBus.
* To enforce separation, create a new `EWIContext` and copy data.

### UTIL (complete function list)

* **Data/Field Manipulation**

  * `moveDataToList`, `copyDataToList`, `moveFieldToListAllRows`, `copyFieldToListAllRows`, `copyFieldFromList`, `addValueToListAllRows`, `combineList`, `filterList`, `joinLeftList`
  * `makeField`, `makeDataList`, `setFieldValue`, `setFieldInList`, `removeFieldInList`, `renameField`, `appendValue`
* **DataList/Collection**

  * `getListCount`, `getDataInList`, `getCollectionInList`, `getDataListInDataList`, `getListInDataList`, `addMapToDataList`, `insertMapToDataList`, `setMapToDataList`, `getMapInDataList`, `getTwoLevelDataList`, `setTwoLevelDataList`, `addDataListToDataList`, `moveDataListToDataList`
* **Map/JSON**

  * `getMapValue`, `setMapValue`, `updateSubFieldInMapValue`, `getSubFieldInMapValue`, `getSubFieldInMapValueAsString`
* **Type Conversion/Validation**

  * `getFieldValue`, `setFieldValue`, `getDoubleValue`, `setDoubleValue`, `getIntegerValue`, `setIntegerValue`, `getLongValue`, `setLongValue`, `getBigDecimalValue`, `getBigIntegerValue`, `getBooleanValue`, `setBooleanValue`, `getFloatValue`, `setFloatValue`
  * `isBlankField`, `isBlankFieldTrim`, `isBlankFieldInDataList`, `isDateValue`
* **Date/Time**

  * `convertDateFormat`, `changeTimeFormat`, `changeTimeFormatInList`, `getTimeByFormat`
* **UUID/Counter**

  * `getUUID`, `getUUID8bit`, `getUUID10bit`, `getCountValue`, `setCountValue`, `increaseCount`, `decreaseCount`
* **Sum/Math**

  * `sumFieldInList`, `increaseValue`
* **API/External**

  * `callJsonAPI`, `callSSQAPI`, `callEWIAPI`, `callExternalPlugin`
* **Debug/Print**

  * `printField`, `printAllFields`
* **Property/Config**

  * `getProperty`
* **Others**

  * `convertEWIContextToJson`, `copyJsonToEWIContext`, `copyDocumentContextToEWIContext`, `throwHttpStatus`

### EWIContextData (like a column/field in a table)

* Stores data for a single field (name, value, meta, children).
* Properties:

  * `name`: field name
  * `value`: field value (String)
  * `metaField`: meta info (`FieldBean`)
  * `fieldType`: field type (`FieldTypeBean`)
  * `children`: if a DataList, `children` holds `List<EWIContextCollection>`
* Methods:

  * `getName()` / `setName(String)`
  * `getValue()` / `setValue(String)`
  * `getMetaField()` / `setMetaField(FieldBean)`
  * `getFieldType()` / `setFieldType(FieldTypeBean)`
  * `getChildren()` / `setChildren(List)`
  * `addChildren(EWIContextCollection)` / `addChildren(Collection)`
  * `getChildrenAt(int index)`
  * `isHidden()`, `setHidden(boolean)`
  * `isReadOnly()`, `setReadOnly(boolean)`
  * `isFocused()`, `setFocused(boolean)`
  * `getLength()`, `setLength(int)`
  * `getValueAsInteger()`, `getValueAsLong()`, `getValueAsDouble()`, `getValueAsDate()`
  * `isChildrenEmpty()`, `childrenSize()`, `containsChildren(Object)`, `removeChildren(Object)`, `iteratorChildren()`
  * `toString()`

### EWIContextCollection (like a row in a table)

* Stores one row of a DataList (`Map<String, EWIContextData>`).
* Methods:

  * `addContextData(String name, EWIContextData dataField)`: add/update a field in the row
  * `getContextData(String name)`: get a field by name
  * `containsKey(Object key)`: whether the row contains a given field
  * `isEmpty()`: whether the row is empty
  * `keySet()`: list all field names in the row
  * `size()`: number of fields in the row
  * `putAll(Map dataItems)`: put multiple fields at once
  * `getAll()`: return `Map<String, EWIContextData>` for the row

---

## Example Usage of `EWIContext` output

```java
// Read a value from output
EWIContextData data = output.getContextData("fieldName");
String value = data != null ? data.getValue() : null;

// Put a new value into output
EWIContextData newData = new EWIContextData();
newData.setName("newField");
newData.setValue("someValue");
output.setContextData(newData);

// Put a DataList into output
EWIContextData listData = new EWIContextData();
listData.setName("myList");
listData.setMetaField(...); // set meta as a collection
output.setContextData(listData);
```

---

## Notes

* `EWIContext` = the “main table” (input/output) that holds all Node code data.
* `EWIContextData` = a “column/field” in the table (stores each field’s value).
* `EWIContextCollection` = a “row” in a DataList (each row is one data record).
* `DataList` = a field with `children` (`List<EWIContextCollection>`).
* `UTIL`/`UTILExtension` = functions for handling context, DataList, Map, APIs, etc.

---

## 3.2 Reading/Writing IBM i (AS/400) Screens with Node Code

This section summarizes how to read and write values on IBM i (AS/400) screens using Node code supported by iConnector, with injected functions such as `getScreenString`, `getService().setText`, and `execute` (for key input).

### Reading values from the screen

* Use `getScreenString(row, col, length)` to read values at the desired position.

Example:

```java
String productCode = getScreenString(8, 10, 6); // Read 6 characters at row 8, column 10
```

### Writing values to the screen

* Use `getService().setText(row, col, value)` to write values at the desired position.

Example:

```java
getService().setText(12, 15, "ABC123"); // Write "ABC123" at row 12, column 15
// Press Enter to confirm
execute(null, HostConstants.KEY_ENTER_STR);
```

### Reading/writing multiple fields on one screen

* You can loop to read or write multiple fields in one go.

Example:

```java
// Read values for all rows in a table (e.g., 10 rows)
for (int row = 5; row <= 14; row++) {
    String item = getScreenString(row, 10, 8);
    // Process item as needed
}
// Write values to multiple rows
for (int row = 5; row <= 14; row++) {
    getService().setText(row, 20, "Y");
}
execute(null, HostConstants.KEY_ENTER_STR);
```

### Reading multiple pages (Loop + Page Down)

* Use a loop to read page by page and press Page Down to continue.

Example:

```java
PageLoop: for (int pageNo = 1; pageNo <= maxPage; pageNo++) {
    for (int row = startRow; row <= endRow; row += rowHeight) {
        String value = getScreenString(row, col, length);
        // Validate/process value
    }
    // If the screen indicates more pages, go to the next page
    if (checkScreen("+", 24, 80, 1)) {
        execute(null, HostConstants.KEY_PAGEDOWN_STR);
    } else {
        break PageLoop;
    }
}
```

### Cautions

* Verify row/col positions match the actual IBM i screen.
* The read/write length must not exceed the screen bounds.
* After writing, always check results or any on-screen messages.

### Full sample code (Selection List)

* See full examples in: `CreateOrder_sub_createProduct.java`, `UpdateChangeAndRelease.java`, `InquiryChangeAndRelease.java`.

---

## 3.3 Error Handling and Stopping the Flow

In iConnector, to stop the flow and return an error message, use `EWIException`.

### Stopping the Flow & Showing an Error Message

When an error is detected and you want to stop the flow and show an error message, there are two approaches:

#### 1. Read the Error Message from the Screen

```java
String errorMessage = getScreenString(24, 2, 79);

EWIException error = new EWIException(errorMessage);
error.setResponseCode(EWIContext.RESPONSE_CODE_INCORRECT_DATA);
throw error;
```

#### 2. Set the Error Message Yourself

```java
EWIException error = new EWIException("Invalid data. Please re-enter.");
error.setResponseCode(EWIContext.RESPONSE_CODE_INCORRECT_DATA);
throw error;
```

### Usage Details

#### 1. Read the error message from the screen

```java
String errorMessage = getScreenString(24, 2, 79);
```

* Reads the IBM i/AS400 on-screen error message at row 24, column 2, length 79.
* This is the standard position where the host displays error messages.

#### 2. Create `EWIException`

```java
EWIException error = new EWIException(errorMessage);
```

* Creates an exception object with the error message read from the screen.

#### 3. Set the response code

```java
error.setResponseCode(EWIContext.RESPONSE_CODE_INCORRECT_DATA);
```

* Sets a response code to indicate the error type.
* Available response codes include (examples):

  * `EWIContext.RESPONSE_CODE_INCORRECT_DATA`: incorrect data

#### 4. Throw the exception

```java
throw error;
```

* Stops the flow and returns the error message.

### Example in Real Context

```java
// Method 1: read error message from the screen
if (checkScreen("Error", 24, 1, 5)) {
    String errorMessage = getScreenString(24, 2, 79);
    logger.error("Error occurred: " + errorMessage);
    
    EWIException error = new EWIException(errorMessage);
    error.setResponseCode(EWIContext.RESPONSE_CODE_INCORRECT_DATA);
    throw error;
}

// Method 2: set your own error message – validate input
EWIContextData productCode = input.getContextData("PRODUCT_CODE");
if (productCode == null || StringUtil.isBlank(productCode.getValue())) {
    String errorMessage = "Please specify a product code.";
    logger.error(errorMessage);
    
    EWIException error = new EWIException(errorMessage);
    error.setResponseCode(EWIContext.RESPONSE_CODE_VALIDATION_ERROR);
    throw error;
}

// Method 2: set your own error message – check processing result
if (!isProcessSuccess) {
    String errorMessage = "Processing failed. Please verify the data and try again.";
    logger.error(errorMessage);
    
    EWIException error = new EWIException(errorMessage);
    error.setResponseCode(EWIContext.RESPONSE_CODE_BUSINESS_ERROR);
    throw error;
}

// Example business rule validation
EWIContextData quantity = input.getContextData("QUANTITY");
if (quantity != null && !StringUtil.isBlank(quantity.getValue())) {
    try {
        int qty = Integer.parseInt(quantity.getValue());
        if (qty <= 0) {
            EWIException error = new EWIException("Quantity must be greater than 0.");
            error.setResponseCode(EWIContext.RESPONSE_CODE_VALIDATION_ERROR);
            throw error;
        }
        if (qty > 999) {
            EWIException error = new EWIException("Quantity must not exceed 999 pieces.");
            error.setResponseCode(EWIContext.RESPONSE_CODE_VALIDATION_ERROR);
            throw error;
        }
    } catch (NumberFormatException e) {
        EWIException error = new EWIException("Quantity must be a number.");
        error.setResponseCode(EWIContext.RESPONSE_CODE_VALIDATION_ERROR);
        throw error;
    }
}
```

### Cautions

* Use `logger.error()` to record the error message before throwing the exception.
* Ensure the error message read from the screen is not null or empty.
* Choose an appropriate response code for the error type.
* Throwing `EWIException` stops the flow immediately—use with care.

### Example: Check the Error Message Before Use

```java
String errorMessage = getScreenString(24, 2, 79);

// Check whether there is an error message
if (errorMessage != null && !StringUtil.isBlank(errorMessage.trim())) {
    logger.error("Screen error detected: " + errorMessage);
    
    EWIException error = new EWIException(errorMessage.trim());
    error.setResponseCode(EWIContext.RESPONSE_CODE_INCORRECT_DATA);
    throw error;
} else {
    logger.info("No error message found on screen");
}
```

---

## 3.2.1 Notes on Screen Coordinates and Screen Size (IBM i/AS400)

* **Screen coordinates:** referenced by row and column, starting at `row:1, column:1` (top-left corner).
* **Default screen size:** if unspecified, use `24 rows × 80 columns`.
* **Code page:** standard is `cp838`.

Examples for reading/writing on screen:

```java
// Read 10 characters at row 1, column 1 (top-left)
String value = getScreenString(1, 1, 10);

// Write "ABC" at row 1, column 1
getService().setText(1, 1, "ABC");
```

### Setting the Cursor Position with `setCursor(row, col)`

* On some IBM i (AS/400) screens, key actions (Enter, F3, etc.) only take effect when the cursor is at a specific position.
* Use `setCursor(row, col);` to move the cursor before pressing a key.

Example:

```java
// Move the cursor to row 10, column 5
setCursor(10, 5);

// Press Enter at that position
execute(null, HostConstants.KEY_ENTER_STR);
```

**Cautions:**

* Make sure the row/col matches what the IBM i screen expects.
* If you don’t `setCursor` correctly, the action may have no effect or the flow may fail.

---

### Example: Filling Comments on the “Work with Order Comments” Screen

This example shows how to fill the comments list on the IBM i “Work with Order Comments” screen using data from the `CusCommentList` DataList.

**Input data format:**

```json
"CusCommentList": [
  {
    "ID": "e959d7a763f6446a85e9715af0c5052c",
    "OrderNo": "",
    "Comment": "Total price includes warehouse fee, 600 THB per ton",
    "FlagInv": "N",
    "FlagPick": "N",
    "FlagAck": "N",
    "FlagBol": "N",
    "FlagPak": "N"
  }
]
```

**IBM i screen details:**

* The “Work with Order Comments” screen starts the comment field at row 11.
* Each row has a comment field (column 2) and flag fields on the right (column 79).
* It shows 10 items per page (use Page Down to go to the next page).

**Sample code:**

```java
EWIContextData cusCommentList = input.getContextData("CusCommentList");

if (cusCommentList != null && cusCommentList.getChildren() != null) {
    List<Object> children = cusCommentList.getChildren();
    int itemsPerPage = 10;
    int startRow = 11;
    int currentRow = startRow;
    int commentCol = 2;
    int flagCol = 79;
    int totalPages = (int) Math.ceil(children.size() / (double) itemsPerPage);

    logger.info("Start clearing and filling CusCommentList with a total of " + children.size() + " items");
    logger.info("totalPages: " + totalPages);
    
    // Fill new data
    currentRow = startRow;
    int index = 0;
    for (int i = 0; i < children.size(); i++) {
        Object obj = children.get(i);
        if (!(obj instanceof EWIContextCollection)) continue;
        EWIContextCollection row = (EWIContextCollection) obj;

        String comment = row.getContextData("Comment") != null ? row.getContextData("Comment").getValue() : "";
        String flagInv = row.getContextData("FlagInv") != null ? row.getContextData("FlagInv").getValue() : "N";

        getService().setText(currentRow, commentCol, comment);
        getService().setText(currentRow, flagCol, flagInv);

        logger.info("Filled Comment at row " + currentRow + ": \"" + comment + "\" (FlagInv=" + flagInv + ")");

        currentRow++;
        index++;

        if (index % itemsPerPage == 0 && index < children.size()) {
            execute(null, HostConstants.KEY_PAGEDOWN_STR);
            currentRow = startRow;
        }
    }
    logger.info("All comments saved successfully");
} else {
    logger.warn("CusCommentList has no data to fill.");
}
```

**Explanation:**

* Reads data from the `CusCommentList` DataList and writes it to the “Work with Order Comments” screen.
* Uses `getService().setText` to fill comments and flag values.
* Handles pagination by pressing Page Down when a page is full.
* Logs every step for traceability and troubleshooting.

**Cautions:**

* Ensure `startRow`, `commentCol`, and `flagCol` match the actual screen layout.
* Verify the current screen is indeed “Work with Order Comments” using `checkScreen`.
* Ensure the comment length does not exceed the screen’s supported length (truncate or handle if necessary).
