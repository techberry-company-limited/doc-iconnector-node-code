# Node Code Reference สำหรับ Developer (Low Code Platform) - iConnector

## 1. เป้าหมายของเอกสารนี้
- ให้ developer และ AI Assistant สามารถเขียน, ตรวจสอบ, และแนะนำ Node code ได้อย่างถูกต้อง ปลอดภัย และ maintainable
- ใช้เป็นคู่มือกลางที่ "เพียงพอในตัวเอง" ไม่ต้องอ้างอิงเอกสารอื่น
- **ระบบที่ iConnector ใช้งานร่วมคือ IBM i (AS/400)**
- **การอ้างอิงตำแหน่งบนหน้าจอใช้รูปแบบ row (แถว) และ column (คอลัมน์) โดยจุดเริ่มต้นคือ row:1, column:1 (มุมบนซ้ายของหน้าจอ)**
- **หากไม่ได้ระบุขนาดหน้าจอ จะใช้ค่าเริ่มต้น 24 แถว x 80 คอลัมน์ (code page: cp838)**

---

## 2. ข้อจำกัดและแนวปฏิบัติ (MUST READ)
- **ห้าม import library หรือ class ใดๆ เพิ่มเติม** (ทุกอย่างต้องใช้จากระบบเท่านั้น)
- ห้ามสร้างคลาส/เมธอดใหม่เอง (ใช้เฉพาะที่ระบบเตรียมไว้)
- ใช้ได้เฉพาะ object/function ที่ระบบ inject ให้ เช่น `input`, `logger`, `UTIL`, `UTILExtension`, `EWIContextData`, `EWIContextCollection`
- หลีกเลี่ยงการแก้ไขโครงสร้างหลักของ flow หรือ context data ที่ระบบกำหนด
- ตรวจสอบ null/empty ทุกครั้งก่อนใช้งานข้อมูล (เช่น `contextData == null`, `StringUtil.isBlank(value)`)
- ใช้ logger สำหรับ debug และ trace ทุกจุดสำคัญ
- ทุกฟังก์ชัน/อ็อบเจกต์ที่ใช้ได้จะถูก inject มาให้พร้อมใช้งานใน Node code
- หลีกเลี่ยงการใช้ static variable, global state, หรือ logic ที่มีผลข้างเคียงกับ flow อื่น
- **Node code ต้องใช้ syntax ที่รองรับโดย JDK 1.8 เท่านั้น (เช่น หลีกเลี่ยง syntax ของ Java 9+ เช่น var, stream API บางอย่าง, lambda expression ที่ซับซ้อน, ฯลฯ)**

---

## 3. รายชื่อ Helper/Function ที่ใช้ได้ (พร้อมตัวอย่างและคำอธิบาย)

### input (อ่าน context data)
- `input.getContextData(fieldName)` : ดึง context data ตามชื่อ field (return EWIContextData)
- `input.getContextData(fieldName).getValue()` : ดึงค่าจริงจาก field (String, Number, ฯลฯ)
- `input.getContextData(fieldName).getChildren()` : ดึง DataList (List<EWIContextCollection>)
- ตัวอย่าง:
  ```java
  EWIContextData data = input.getContextData("NBROL");
  if (data != null && !StringUtil.isBlank(data.getValue())) {
      String value = data.getValue();
  }
  ```

### logger (log ข้อมูล)
- `logger.info(msg)` : log ข้อมูลทั่วไป
- `logger.warn(msg)` : log คำเตือน
- `logger.error(msg)` : log ข้อผิดพลาด
- ตัวอย่าง:
  ```java
  logger.info("Start Node: Create Attribute List");
  logger.warn("Field X is null");
  logger.error("Unexpected error: " + e.getMessage());
  ```

### UTIL (ฟังก์ชันอรรถประโยชน์)
- `UTIL.addMapToDataList(listName, mapObj)` : เพิ่ม map เข้า DataList
- `UTIL.getMapValue(fieldName)` : อ่านค่า Map (JSON) จาก field (return Map<String, Object>)
- `UTIL.setMapValue(fieldName, mapObj)` : เขียนค่า Map กลับเข้า field
- `UTIL.changeTimeFormatInList(listName, fieldName, fromFormat, toFormat, isAllRow)` : เปลี่ยนรูปแบบวันที่ใน DataList
- `UTIL.getUUID()` : สร้าง UUID 32 หลัก (String)
- `UTIL.getUUID8bit()` : สร้าง UUID 8 หลัก (String)
- `UTIL.getDataListInDataList(mainList, subListField, index)` : ดึง DataList ซ้อน (List<EWIContextCollection>)
- ตัวอย่าง:
  ```java
  Map<String, Object> map = UTIL.getMapValue("myMapField");
  if (map != null) {
      Object v = map.get("subKey");
  }
  UTIL.setMapValue("myMapField", map);
  String uuid = UTIL.getUUID();
  ```

### UTILExtension (ฟังก์ชันเสริม)
- ฟังก์ชันแปลงข้อมูล, validate, จัดการ string/number (ดูชื่อ method ใน helper/UTILExtension.java)
- ตัวอย่าง:
  ```java
  String result = UTILExtension.someUtilityMethod(args);
  ```

### EWIContextData, EWIContextCollection (อ่าน/เขียน context data)
- ใช้สำหรับสร้าง/แก้ไข field หรือ row ใน DataList
- ตัวอย่าง: สร้าง field ใหม่ใน row
  ```java
  EWIContextData statusField = new EWIContextData();
  statusField.setName("status");
  statusField.setValue("fail");
  row.addContextData("status", statusField);
  ```
- อ่านค่าจาก row:
  ```java
  String value = row.getContextData("fieldName").getValue();
  ```

---

## 3.1 การ Key In พิเศษ (Special Key Input)

ในระบบ iConnector การสั่ง "Key In" หรือกดปุ่มพิเศษ (เช่น Enter, F3, PageDown, PF12 ฯลฯ) สามารถทำได้โดยใช้เมธอด `execute()` หรือ `execute(null, keyString)` ที่ระบบเตรียมไว้

### วิธีใช้งาน

- `execute();`  
  ใช้สำหรับกด Enter (หรือยืนยันข้อมูลบนหน้าจอปัจจุบัน)
- `execute(null, HostConstants.KEY_ENTER_STR);`  
  ใช้สำหรับกด Enter (เหมือน execute();)
- `execute(null, HostConstants.KEY_F1_STR);`  
  ใช้สำหรับกดปุ่ม F1
- `execute(null, HostConstants.KEY_F2_STR);`  
  ใช้สำหรับกดปุ่ม F2
- `execute(null, HostConstants.KEY_F3_STR);`  
  ใช้สำหรับกดปุ่ม F3
- `execute(null, HostConstants.KEY_F4_STR);`  
  ใช้สำหรับกดปุ่ม F4
- `execute(null, HostConstants.KEY_F5_STR);`  
  ใช้สำหรับกดปุ่ม F5
- `execute(null, HostConstants.KEY_F6_STR);`  
  ใช้สำหรับกดปุ่ม F6
- `execute(null, HostConstants.KEY_F7_STR);`  
  ใช้สำหรับกดปุ่ม F7
- `execute(null, HostConstants.KEY_F8_STR);`  
  ใช้สำหรับกดปุ่ม F8
- `execute(null, HostConstants.KEY_F9_STR);`  
  ใช้สำหรับกดปุ่ม F9
- `execute(null, HostConstants.KEY_F10_STR);`  
  ใช้สำหรับกดปุ่ม F10
- `execute(null, HostConstants.KEY_F11_STR);`  
  ใช้สำหรับกดปุ่ม F11
- `execute(null, HostConstants.KEY_F12_STR);`  
  ใช้สำหรับกดปุ่ม F12

- `execute(null, HostConstants.KEY_PF1_STR);`  
  ใช้สำหรับกดปุ่ม PF1
- `execute(null, HostConstants.KEY_PF2_STR);`  
  ใช้สำหรับกดปุ่ม PF2
- `execute(null, HostConstants.KEY_PF3_STR);`  
  ใช้สำหรับกดปุ่ม PF3
- `execute(null, HostConstants.KEY_PF4_STR);`  
  ใช้สำหรับกดปุ่ม PF4
- `execute(null, HostConstants.KEY_PF5_STR);`  
  ใช้สำหรับกดปุ่ม PF5
- `execute(null, HostConstants.KEY_PF6_STR);`  
  ใช้สำหรับกดปุ่ม PF6
- `execute(null, HostConstants.KEY_PF7_STR);`  
  ใช้สำหรับกดปุ่ม PF7
- `execute(null, HostConstants.KEY_PF8_STR);`  
  ใช้สำหรับกดปุ่ม PF8
- `execute(null, HostConstants.KEY_PF9_STR);`  
  ใช้สำหรับกดปุ่ม PF9
- `execute(null, HostConstants.KEY_PF10_STR);`  
  ใช้สำหรับกดปุ่ม PF10
- `execute(null, HostConstants.KEY_PF11_STR);`  
  ใช้สำหรับกดปุ่ม PF11
- `execute(null, HostConstants.KEY_PF12_STR);`  
  ใช้สำหรับกดปุ่ม PF12
- `execute(null, HostConstants.KEY_PF13_STR);`  
  ใช้สำหรับกดปุ่ม PF13
- `execute(null, HostConstants.KEY_PF14_STR);`  
  ใช้สำหรับกดปุ่ม PF14
- `execute(null, HostConstants.KEY_PF15_STR);`  
  ใช้สำหรับกดปุ่ม PF15
- `execute(null, HostConstants.KEY_PF16_STR);`  
  ใช้สำหรับกดปุ่ม PF16
- `execute(null, HostConstants.KEY_PF17_STR);`  
  ใช้สำหรับกดปุ่ม PF17
- `execute(null, HostConstants.KEY_PF18_STR);`  
  ใช้สำหรับกดปุ่ม PF18
- `execute(null, HostConstants.KEY_PF19_STR);`  
  ใช้สำหรับกดปุ่ม PF19
- `execute(null, HostConstants.KEY_PF20_STR);`  
  ใช้สำหรับกดปุ่ม PF20
- `execute(null, HostConstants.KEY_PF21_STR);`  
  ใช้สำหรับกดปุ่ม PF21
- `execute(null, HostConstants.KEY_PF22_STR);`  
  ใช้สำหรับกดปุ่ม PF22
- `execute(null, HostConstants.KEY_PF23_STR);`  
  ใช้สำหรับกดปุ่ม PF23
- `execute(null, HostConstants.KEY_PF24_STR);`  
  ใช้สำหรับกดปุ่ม PF24

- `execute(null, HostConstants.KEY_PAGEDOWN_STR);`  
  ใช้สำหรับกด PageDown
- `execute(null, HostConstants.KEY_PAGEUP_STR);`  
  ใช้สำหรับกด PageUp
- `execute(null, HostConstants.KEY_HOME_STR);`  
  ใช้สำหรับกด Home
- `execute(null, HostConstants.KEY_END_STR);`  
  ใช้สำหรับกด End
- `execute(null, HostConstants.KEY_INSERT_STR);`  
  ใช้สำหรับกด Insert
- `execute(null, HostConstants.KEY_DELETE_STR);`  
  ใช้สำหรับกด Delete
- `execute(null, HostConstants.KEY_TAB_STR);`  
  ใช้สำหรับกด Tab
- `execute(null, HostConstants.KEY_BACKSPACE_STR);`  
  ใช้สำหรับกด Backspace
- `execute(null, HostConstants.KEY_ESC_STR);`  
  ใช้สำหรับกด Escape (ESC)
- `execute(null, HostConstants.KEY_UP_STR);`  
  ใช้สำหรับกดลูกศรขึ้น
- `execute(null, HostConstants.KEY_DOWN_STR);`  
  ใช้สำหรับกดลูกศรลง
- `execute(null, HostConstants.KEY_LEFT_STR);`  
  ใช้สำหรับกดลูกศรซ้าย
- `execute(null, HostConstants.KEY_RIGHT_STR);`  
  ใช้สำหรับกดลูกศรขวา

> **หมายเหตุ:**  
> - ค่าคีย์พิเศษต่างๆ จะอยู่ในคลาส `HostConstants`  
> - สามารถดูชื่อคีย์ทั้งหมดได้จาก HostConstants (เช่น KEY_F3_STR, KEY_PF3_STR, KEY_PAGEDOWN_STR, KEY_PF12_STR ฯลฯ)
> - บางระบบอาจรองรับเฉพาะบางคีย์ ขึ้นกับหน้าจอและระบบ Host

### ตัวอย่าง

```java
// กด Enter (ปกติ)
execute();
execute(null, HostConstants.KEY_ENTER_STR);

// กด F3
execute(null, HostConstants.KEY_F3_STR);

// กด PF12
execute(null, HostConstants.KEY_PF12_STR);

// กด PageDown
execute(null, HostConstants.KEY_PAGEDOWN_STR);

// กดลูกศรขึ้น
execute(null, HostConstants.KEY_UP_STR);

// กด ESC
execute(null, HostConstants.KEY_ESC_STR);
```

### ตัวอย่างการใช้งานใน Flow

```java
if (checkScreen(OIS100_A)) {
    // กรอกข้อมูลในฟิลด์ต่างๆ
    getService().setText(4, 18, "3");
    setContextData(input.getContextData("coNo"));

    // กด Enter เพื่อยืนยัน
    execute();

    // กด F3 เพื่อย้อนกลับ
    execute(null, HostConstants.KEY_F3_STR);

    // กด PageDown เพื่อเลื่อนหน้าจอ
    execute(null, HostConstants.KEY_PAGEDOWN_STR);
}
```

> **ข้อควรระวัง:**  
> - ควรตรวจสอบหน้าจอ (checkScreen) ก่อน execute ทุกครั้ง  
> - การ execute ซ้ำโดยไม่เปลี่ยนแปลงข้อมูลอาจทำให้ flow ผิดพลาด  
> - ใช้ logger.info ทุกครั้งที่มีการ execute เพื่อ debug

---

## 4. ตัวอย่าง Use-case สำคัญ (Best Practice พร้อมโค้ด)

### 4.1 แปลง field เป็น array (Attribute List)
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

### 4.2 อ่าน/เขียนค่า Map
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

### 4.3 เปลี่ยนรูปแบบวันที่ใน DataList
```java
UTIL.changeTimeFormatInList("orderList", "orderDate", "yyyyMMdd", "dd/MM/yyyy", false);
```

### 4.4 สร้าง UUID
```java
String uuid = UTIL.getUUID();
String uuid8 = UTIL.getUUID8bit();
```

### 4.5 ดึง DataList ซ้อน
```java
List<EWIContextCollection> subList = UTIL.getDataListInDataList("mainList", "subListField", 0);
logger.info("SubList size: " + (subList != null ? subList.size() : 0));
```

### 4.6 เพิ่ม column ใน DataList

> **ข้อควรระวัง:**  
> - ก่อนวนลูป children ของ DataList ควรตรวจสอบชนิด (instanceof) ว่าเป็น EWIContextCollection จริง  
> - การเพิ่ม context data ใหม่ใน row ให้ใช้ method ที่ class รองรับ (เช่น addContextData หรือ setContextData)  
> - ตัวอย่างนี้รองรับกรณีที่ children อาจไม่ใช่ EWIContextCollection ทั้งหมด

```java
EWIContextData productList = input.getContextData("productList");
if (productList != null && productList.getChildren() != null) {
    for (Object obj : productList.getChildren()) {
        if (!(obj instanceof EWIContextCollection)) continue;
        EWIContextCollection row = (EWIContextCollection) obj;

        // ตัวอย่างการเพิ่ม field ใหม่ชื่อ "status"
        EWIContextData statusField = row.getContextData("status");
        if (statusField == null) {
            statusField = new EWIContextData();
            statusField.setName("status");
            row.addContextData("status", statusField); // หรือใช้ setContextData ถ้ามี
        }
        statusField.setValue("fail");
    }
} else {
    logger.warn("productList is null or has no children.");
}
```

---

### หมายเหตุเพิ่มเติมเกี่ยวกับการวนลูปและอัปเดต DataList

- หาก children ของ DataList ไม่ใช่ EWIContextCollection ทั้งหมด ให้ตรวจสอบชนิดก่อน cast
- การเพิ่ม context data ใหม่ใน row ให้ใช้ method ที่ class นั้นรองรับ (addContextData, setContextData หรือ putContextData)
- ตัวอย่างการเช็คและอัปเดต field ใน row อย่างปลอดภัย:

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
            row.addContextData("flag", flagField); // หรือใช้ setContextData ถ้ามี
        }
        flagField.setValue(flagValue);
    }
} else {
    logger.warn("productList is null or has no children.");
}
```

**สรุป:**  
- ตรวจสอบชนิดของ children ก่อน cast  
- ใช้ method ที่ class รองรับในการเพิ่ม context data  
- ตรวจสอบ null/empty ทุกครั้งก่อนใช้งานข้อมูล  
- ใช้ logger สำหรับ debug ทุกจุดสำคัญ

---

## 5. โครงสร้างการทำงานของ Node code (generatecode)

- Node code จะถูก generate และ execute ใน flow ตามที่กำหนดใน generatecode
- ทุกฟังก์ชัน/อ็อบเจกต์ที่ใช้ได้จะถูก inject มาให้พร้อมใช้งาน
- หลีกเลี่ยงการแก้ไข context data หลักที่ระบบใช้ในการ flow
- สามารถ debug/trace ได้ด้วย logger
- โค้ดต้อง stateless (ไม่เก็บ state ข้ามรอบ execution)

---

### สรุปหลักการ DataBus (EWIContext) และความสัมพันธ์ระหว่าง input / output

- **DataBus** คืออ็อบเจกต์หลักที่ใช้เก็บและส่งผ่านข้อมูลระหว่าง Node code แต่ละขั้นตอนใน flow ของ iConnector
- DataBus มีโครงสร้างเหมือน Map<String, EWIContextData> โดยแต่ละ field (หรือ DataList) จะถูกเก็บในรูปแบบ EWIContextData
- ข้อมูลใน DataBus สามารถเป็นได้ทั้ง field เดี่ยว (String, Number ฯลฯ) หรือ DataList (List<EWIContextCollection>)
- ทุก Node code จะรับ **input** (EWIContext) และส่งผลลัพธ์ผ่าน **output** (EWIContext) ซึ่งมักจะเป็นอ็อบเจกต์เดียวกัน (เช่น dataBus)
- การแก้ไขข้อมูลใน input (เช่น เพิ่ม/แก้ไข field หรือ DataList) จะมีผลกับ output ทันที ถ้า input กับ output อ้างอิง object เดียวกัน (pass by reference)
- หากต้องการให้ output ไม่ได้รับผลกระทบจาก input ต้องมีการ clone/copy context ใหม่ (แต่ในระบบ generatecode ส่วนใหญ่จะใช้ object เดียวกัน)

#### ความสัมพันธ์ input และ output

- **input**: ใช้สำหรับอ่านข้อมูล context ที่ถูกส่งเข้ามาใน Node code
- **output**: ใช้สำหรับเขียนหรืออัปเดตข้อมูล context ที่จะถูกส่งต่อไปยัง Node ถัดไป หรือเป็นผลลัพธ์สุดท้ายของ flow
- ใน flow ปกติ input และ output จะอ้างอิง DataBus object เดียวกัน (เช่น processFlow(dataBus, dataBus))
- ดังนั้น การแก้ไขข้อมูลผ่าน input หรือ output จะมีผลกับ DataBus เดียวกัน (ข้อมูลที่แก้ไขจะถูกส่งต่อไปใน flow)
- ตัวอย่างเช่น การเพิ่ม column ใน DataList ของ input จะทำให้ output เห็นข้อมูลที่เพิ่มทันที

**ข้อควรระวัง:**  
- หลีกเลี่ยงการแก้ไข context data หลักที่ระบบใช้ในการ flow โดยไม่จำเป็น  
- ตรวจสอบ null/empty ทุกครั้งก่อนใช้งานข้อมูล  
- หากต้องการแยก input/output ให้ชัดเจน ต้องสร้าง EWIContext ใหม่และ copy ข้อมูลเอง (แต่ระบบ generatecode มักไม่ใช้ pattern นี้)

**สรุป:**  
- DataBus (EWIContext) คือศูนย์กลางข้อมูลของ Node code  
- input/output มักเป็น object เดียวกัน  
- การแก้ไขข้อมูลใน input จะมีผลกับ output ทันที (pass by reference)  
- ใช้สำหรับส่งผ่านข้อมูลระหว่าง Node/Flow อย่างต่อเนื่อง

---

## 6. แนวทางการ debug และตรวจสอบคุณภาพ
- ใช้ logger.info/warn/error ในทุกจุดสำคัญ (input, output, error, decision)
- ตรวจสอบค่าทุก field ก่อนใช้งาน (null/empty)
- ทดสอบ use-case ตามตัวอย่างก่อน deploy
- หากเกิด error ให้ log รายละเอียดและ context data ที่เกี่ยวข้อง
- หลีกเลี่ยงการ swallow error (ต้อง log ทุก error ที่เกิดขึ้น)

---

## 7. Checklist สำหรับ Developer/AI Assistant
- [ ] หลีกเลี่ยง import หรือสร้างคลาส/เมธอดใหม่
- [ ] ใช้เฉพาะ Helper/Function ที่ระบบเตรียมไว้
- [ ] ตรวจสอบ null/empty ทุกครั้ง
- [ ] ใช้ logger สำหรับ debug
- [ ] ทดสอบ use-case ตามตัวอย่างก่อน deploy
- [ ] อ่าน/อ้างอิงตัวอย่างจาก HowToWriteNodeCode.md และไฟล์ใน generatecode/
- [ ] หากไม่แน่ใจ ให้สอบถามทีมพัฒนา
- [ ] โค้ดต้อง stateless และไม่มีผลข้างเคียงกับ flow อื่น

---

## 8. Reference และแหล่งข้อมูลเพิ่มเติม
- ตัวอย่างและแนวปฏิบัติ: `HowToWriteNodeCode.md`
- รายละเอียด Helper/Function: โฟลเดอร์ `helper/`
- ตัวอย่าง flow จริง: โฟลเดอร์ `generatecode/`
- สอบถามทีมพัฒนา: [ช่องทางติดต่อ]

> **หมายเหตุ:** เอกสารนี้ออกแบบให้ AI หรือ Assistant สามารถใช้เป็นคู่มือกลางในการแนะนำ/ตรวจสอบ/แก้ไข Node code ได้ทันที โดยไม่ต้องอ้างอิงเอกสารอื่น หากต้องการรายละเอียดเพิ่มเติมของ Helper/Function หรือ use-case เฉพาะ สามารถดูไฟล์ `HowToWriteNodeCode.md` และโค้ดในโฟลเดอร์ `generatecode/` หรือสอบถามทีมพัฒนาได้ทันที

---

## 9. โครงสร้างและรายละเอียดเชิงลึกของ EWIContext, UTIL, EWIContextData, EWIContextCollection

### EWIContext (input/output)
- อ็อบเจกต์หลักที่ใช้เก็บข้อมูล context ของ Node code ทั้ง input และ output
- โครงสร้างคล้าย Map<String, EWIContextData>
- ใช้สำหรับอ่าน/เขียน field, DataList, และข้อมูล meta อื่นๆ
- เมธอดสำคัญ:
  - `getContextData(String fieldName)` : ดึง EWIContextData ตามชื่อ field
  - `setContextData(EWIContextData data)` : ใส่/อัปเดต field
  - `removeContextData(String fieldName)` : ลบ field
  - `getAll()` : คืน Map<String, EWIContextData> ทั้งหมด
  - `putAll(Map)` : ใส่ข้อมูลหลาย field พร้อมกัน
  - `getContextData(String[] fieldNames)` : คืน Map เฉพาะ field ที่ต้องการ
  - `getContextMessages()` : ข้อความ/แจ้งเตือนใน context
  - `setResponseCode(String)` / `getResponseCode()` : จัดการ response code
  - `setResponseMessage(String)` / `getResponseMessage()` : จัดการ response message

#### หมายเหตุเกี่ยวกับ input/output

- ในระบบ generatecode ทั่วไป input และ output จะเป็น object เดียวกัน (pass by reference)
- การแก้ไขข้อมูลผ่าน input หรือ output จะมีผลกับ DataBus เดียวกันทันที
- หากต้องการแยก input/output ให้ชัดเจน ต้องสร้าง EWIContext ใหม่และ copy ข้อมูลเอง

### UTIL (ฟังก์ชันที่มีทั้งหมด)
- **Data/Field Manipulation**
  - `moveDataToList`, `copyDataToList`, `moveFieldToListAllRows`, `copyFieldToListAllRows`, `copyFieldFromList`, `addValueToListAllRows`, `combineList`, `filterList`, `joinLeftList`
  - `makeField`, `makeDataList`, `setFieldValue`, `setFieldInList`, `removeFieldInList`, `renameField`, `appendValue`
- **DataList/Collection**
  - `getListCount`, `getDataInList`, `getCollectionInList`, `getDataListInDataList`, `getListInDataList`, `addMapToDataList`, `insertMapToDataList`, `setMapToDataList`, `getMapInDataList`, `getTwoLevelDataList`, `setTwoLevelDataList`, `addDataListToDataList`, `moveDataListToDataList`
- **Map/JSON**
  - `getMapValue`, `setMapValue`, `updateSubFieldInMapValue`, `getSubFieldInMapValue`, `getSubFieldInMapValueAsString`
- **Type Conversion/Validation**
  - `getFieldValue`, `setFieldValue`, `getDoubleValue`, `setDoubleValue`, `getIntegerValue`, `setIntegerValue`, `getLongValue`, `setLongValue`, `getBigDecimalValue`, `getBigIntegerValue`, `getBooleanValue`, `setBooleanValue`, `getFloatValue`, `setFloatValue`
  - `isBlankField`, `isBlankFieldTrim`, `isBlankFieldInDataList`, `isDateValue`
- **Date/Time**
  - `convertDateFormat`, `changeTimeFormat`, `changeTimeFormatInList`, `getTimeByFormat`
- **UUID/Counter**
  - `getUUID`, `getUUID8bit`, `getUUID10bit`, `getCountValue`, `setCountValue`, `increaseCount`, `decreaseCount`
- **Sum/Math**
  - `sumFieldInList`, `increaseValue`
- **API/External**
  - `callJsonAPI`, `callSSQAPI`, `callEWIAPI`, `callExternalPlugin`
- **Debug/Print**
  - `printField`, `printAllFields`
- **Property/Config**
  - `getProperty`
- **อื่นๆ**
  - `convertEWIContextToJson`, `copyJsonToEWIContext`, `copyDocumentContextToEWIContext`, `throwHttpStatus`

### EWIContextData (เหมือน column/field ใน table)
- เก็บข้อมูล 1 field (ชื่อ, ค่า, meta, children)
- Properties:
  - `name` : ชื่อ field
  - `value` : ค่าของ field (String)
  - `metaField` : ข้อมูล meta (FieldBean)
  - `fieldType` : ประเภท field (FieldTypeBean)
  - `children` : ถ้าเป็น DataList จะมี children (List<EWIContextCollection>)
- Methods:
  - `getName()` / `setName(String)`
  - `getValue()` / `setValue(String)`
  - `getMetaField()` / `setMetaField(FieldBean)`
  - `getFieldType()` / `setFieldType(FieldTypeBean)`
  - `getChildren()` / `setChildren(List)`
  - `addChildren(EWIContextCollection)` / `addChildren(Collection)`
  - `getChildrenAt(int index)`
  - `isHidden()`, `setHidden(boolean)`
  - `isReadOnly()`, `setReadOnly(boolean)`
  - `isFocused()`, `setFocused(boolean)`
  - `getLength()`, `setLength(int)`
  - `getValueAsInteger()`, `getValueAsLong()`, `getValueAsDouble()`, `getValueAsDate()`
  - `isChildrenEmpty()`, `childrenSize()`, `containsChildren(Object)`, `removeChildren(Object)`, `iteratorChildren()`
  - `toString()`

### EWIContextCollection (เหมือน row ใน table)
- เก็บข้อมูล 1 row ของ DataList (Map<String, EWIContextData>)
- Methods:
  - `addContextData(String name, EWIContextData dataField)` : เพิ่ม/อัปเดต field ใน row
  - `getContextData(String name)` : ดึง field ตามชื่อ
  - `containsKey(Object key)` : row นี้มี field นี้ไหม
  - `isEmpty()` : row ว่างหรือไม่
  - `keySet()` : รายชื่อ field ทั้งหมดใน row
  - `size()` : จำนวน field ใน row
  - `putAll(Map dataItems)` : ใส่ข้อมูลหลาย field พร้อมกัน
  - `getAll()` : คืน Map<String, EWIContextData> ทั้งหมดใน row

---

## ตัวอย่างการใช้งาน EWIContext output
```java
// อ่านค่าจาก output
EWIContextData data = output.getContextData("fieldName");
String value = data != null ? data.getValue() : null;

// ใส่ค่าใหม่ใน output
EWIContextData newData = new EWIContextData();
newData.setName("newField");
newData.setValue("someValue");
output.setContextData(newData);

// ใส่ DataList ใน output
EWIContextData listData = new EWIContextData();
listData.setName("myList");
listData.setMetaField(...); // set meta เป็น collection
output.setContextData(listData);
```

---

## หมายเหตุ
- EWIContext = เหมือน "ตารางหลัก" (input/output) ใช้เก็บข้อมูลทั้งหมดของ Node code
- EWIContextData = เหมือน "คอลัมน์/ฟิลด์" ในตาราง (เก็บค่าของแต่ละ field)
- EWIContextCollection = เหมือน "แถว (row)" ใน DataList (แต่ละแถวคือ 1 ชุดข้อมูลใน DataList)
- DataList = field ที่มี children (List<EWIContextCollection>)
- UTIL/UTILExtension = ฟังก์ชันจัดการข้อมูล context, DataList, Map, API, ฯลฯ

---

## 3.2 การอ่าน/เขียนหน้าจอ IBM i (AS/400) ด้วย Node code

เอกสารนี้สรุปวิธีอ่านและเขียนค่าบนหน้าจอ IBM i (AS/400) ด้วย Node code ที่ระบบ iConnector รองรับ โดยใช้ฟังก์ชันที่ระบบ inject ให้ เช่น `getScreenString`, `getService().setText`, และ `execute` (สำหรับ key input)

### การอ่านค่าจากหน้าจอ (Read)
- ใช้ `getScreenString(row, col, length)` เพื่ออ่านค่าจากตำแหน่งที่ต้องการบนหน้าจอ
- ตัวอย่าง:
  ```java
  String productCode = getScreenString(8, 10, 6); // อ่านค่า 6 ตัวอักษร ที่แถว 8 คอลัมน์ 10
  ```

### การเขียนค่าลงหน้าจอ (Write)
- ใช้ `getService().setText(row, col, value)` เพื่อเขียนค่าลงไปที่ตำแหน่งที่ต้องการ
- ตัวอย่าง:
  ```java
  getService().setText(12, 15, "ABC123"); // เขียนค่า "ABC123" ที่แถว 12 คอลัมน์ 15
  // กด Enter เพื่อยืนยัน
  execute(null, HostConstants.KEY_ENTER_STR);
  ```

### การอ่าน/เขียนค่าหลายช่องใน 1 หน้า
- สามารถใช้ลูปอ่านหรือเขียนค่าหลายช่องพร้อมกันได้
- ตัวอย่าง:
  ```java
  // อ่านค่าทุกแถวในตาราง (เช่น 10 แถว)
  for (int row = 5; row <= 14; row++) {
      String item = getScreenString(row, 10, 8);
      // ประมวลผล item ตามต้องการ
  }
  // เขียนค่าลงหลายแถว
  for (int row = 5; row <= 14; row++) {
      getService().setText(row, 20, "Y");
  }
  execute(null, HostConstants.KEY_ENTER_STR);
  ```

### การอ่านค่าหลายหน้า (Loop + Page Down)
- ใช้ลูปอ่านทีละหน้าและกด page down เพื่ออ่านข้อมูลหลายหน้า
- ตัวอย่าง:
  ```java
  PageLoop: for (int pageNo = 1; pageNo <= maxPage; pageNo++) {
      for (int row = startRow; row <= endRow; row += rowHeight) {
          String value = getScreenString(row, col, length);
          // ตรวจสอบ/ประมวลผล value
      }
      // ถ้ามี page down ให้กดเพื่อไปหน้าถัดไป
      if (checkScreen("+", 24, 80, 1)) {
          execute(null, HostConstants.KEY_PAGEDOWN_STR);
      } else {
          break PageLoop;
      }
  }
  ```

### ข้อควรระวัง
- ตรวจสอบตำแหน่ง row/col ให้ตรงกับหน้าจอจริงของ IBM i
- ความยาว (length) ที่อ่านหรือเขียนต้องไม่เกินขอบเขตของหน้าจอ
- หลังการเขียนค่า ควรตรวจสอบผลลัพธ์หรือข้อความแจ้งเตือนจากหน้าจอเสมอ

### ตัวอย่างโค้ดเต็ม (Selection List)
- ดูตัวอย่างโค้ดเต็มในไฟล์: `CreateOrder_sub_createProduct.java`, `UpdateChangeAndRelease.java`, `InquiryChangeAndRelease.java`

---

## 3.3 การจัดการ Error และการหยุด Flow

ในระบบ iConnector เมื่อต้องการหยุด flow และแสดง error message ออกไป สามารถใช้ `EWIException` เพื่อจัดการ error ได้

### การหยุด Flow และแสดง Error Message

เมื่อตรวจพบข้อผิดพลาดหรือต้องการหยุด flow และแสดงข้อความ error มีวิธีการ 2 แบบ:

#### 1. อ่าน Error Message จากหน้าจอ
```java
String errorMessage = getScreenString(24, 2, 79);

EWIException error = new EWIException(errorMessage);
error.setResponseCode(EWIContext.RESPONSE_CODE_INCORRECT_DATA);
throw error;
```

#### 2. กำหนด Error Message เอง
```java
EWIException error = new EWIException("ข้อมูลผิดพลาดโปรดระบุใหม่");
error.setResponseCode(EWIContext.RESPONSE_CODE_INCORRECT_DATA);
throw error;
```

### รายละเอียดการใช้งาน

#### 1. การอ่านข้อความ Error จากหน้าจอ
```java
String errorMessage = getScreenString(24, 2, 79);
```
- อ่านข้อความ error จากหน้าจอ IBM i/AS400 ที่แถว 24 คอลัมน์ 2 ความยาว 79 ตัวอักษร
- ตำแหน่งนี้เป็นตำแหน่งมาตรฐานที่ระบบแสดงข้อความ error บนหน้าจอ

#### 2. การสร้าง EWIException
```java
EWIException error = new EWIException(errorMessage);
```
- สร้าง exception object พร้อมข้อความ error ที่อ่านมาจากหน้าจอ

#### 3. การกำหนด Response Code
```java
error.setResponseCode(EWIContext.RESPONSE_CODE_INCORRECT_DATA);
```
- กำหนด response code เพื่อระบุประเภทของ error
- Response Code ที่ใช้ได้:
  - `EWIContext.RESPONSE_CODE_INCORRECT_DATA` : ข้อมูลไม่ถูกต้อง
#### 4. การ Throw Exception
```java
throw error;
```
- หยุด flow และส่ง error message ออกไป

### ตัวอย่างการใช้งานในบริบทจริง

```java
// วิธีที่ 1: อ่าน error message จากหน้าจอ
if (checkScreen("Error", 24, 1, 5)) {
    String errorMessage = getScreenString(24, 2, 79);
    logger.error("Error occurred: " + errorMessage);
    
    EWIException error = new EWIException(errorMessage);
    error.setResponseCode(EWIContext.RESPONSE_CODE_INCORRECT_DATA);
    throw error;
}

// วิธีที่ 2: กำหนด error message เอง - ตรวจสอบข้อมูล input
EWIContextData productCode = input.getContextData("PRODUCT_CODE");
if (productCode == null || StringUtil.isBlank(productCode.getValue())) {
    String errorMessage = "กรุณาระบุรหัสสินค้า";
    logger.error(errorMessage);
    
    EWIException error = new EWIException(errorMessage);
    error.setResponseCode(EWIContext.RESPONSE_CODE_VALIDATION_ERROR);
    throw error;
}

// วิธีที่ 2: กำหนด error message เอง - ตรวจสอบผลลัพธ์การประมวลผล
if (!isProcessSuccess) {
    String errorMessage = "การประมวลผลล้มเหลว กรุณาตรวจสอบข้อมูลและลองใหม่อีกครั้ง";
    logger.error(errorMessage);
    
    EWIException error = new EWIException(errorMessage);
    error.setResponseCode(EWIContext.RESPONSE_CODE_BUSINESS_ERROR);
    throw error;
}

// ตัวอย่างการตรวจสอบเงื่อนไขทางธุรกิจ
EWIContextData quantity = input.getContextData("QUANTITY");
if (quantity != null && !StringUtil.isBlank(quantity.getValue())) {
    try {
        int qty = Integer.parseInt(quantity.getValue());
        if (qty <= 0) {
            EWIException error = new EWIException("จำนวนสินค้าต้องมากกว่า 0");
            error.setResponseCode(EWIContext.RESPONSE_CODE_VALIDATION_ERROR);
            throw error;
        }
        if (qty > 999) {
            EWIException error = new EWIException("จำนวนสินค้าต้องไม่เกิน 999 ชิ้น");
            error.setResponseCode(EWIContext.RESPONSE_CODE_VALIDATION_ERROR);
            throw error;
        }
    } catch (NumberFormatException e) {
        EWIException error = new EWIException("จำนวนสินค้าต้องเป็นตัวเลขเท่านั้น");
        error.setResponseCode(EWIContext.RESPONSE_CODE_VALIDATION_ERROR);
        throw error;
    }
}
```

### ข้อควรระวัง

- ใช้ logger.error() เพื่อบันทึก error message ก่อน throw exception
- ตรวจสอบว่าข้อความ error ที่อ่านจากหน้าจอไม่เป็น null หรือ empty
- เลือก response code ให้เหมาะสมกับประเภทของ error
- การ throw EWIException จะหยุด flow ทันที ดังนั้นต้องใช้อย่างระมัดระวัง

### ตัวอย่างการตรวจสอบ Error Message ก่อนใช้งาน

```java
String errorMessage = getScreenString(24, 2, 79);

// ตรวจสอบว่ามี error message หรือไม่
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

## 3.2.1 หมายเหตุเรื่องตำแหน่งบนหน้าจอและขนาดหน้าจอ (IBM i/AS400)

- **ตำแหน่งบนหน้าจอ**: อ้างอิงด้วย row (แถว) และ column (คอลัมน์) โดยจุดเริ่มต้นคือ `row:1, column:1` (มุมบนซ้าย)
- **ขนาดหน้าจอ default**: หากไม่ได้ระบุ จะใช้ `24 แถว x 80 คอลัมน์`
- **code page**: ใช้ `cp838` เป็นมาตรฐาน
- ตัวอย่างการอ่าน/เขียนค่าบนหน้าจอ:
  ```java
  // อ่านค่า 10 ตัวอักษร ที่แถว 1 คอลัมน์ 1 (มุมบนซ้าย)
  String value = getScreenString(1, 1, 10);

  // เขียนค่า "ABC" ที่แถว 1 คอลัมน์ 1
  getService().setText(1, 1, "ABC");
  ```

### การกำหนดตำแหน่ง Cursor ด้วย setCursor(row, col)

- ในบางหน้าจอของ IBM i (AS/400) การกด key action (เช่น Enter, F3) จะมีผลเฉพาะเมื่อ cursor อยู่ที่ตำแหน่งที่กำหนด
- ใช้คำสั่ง `setCursor(row, col);` เพื่อย้าย cursor ไปยังตำแหน่งที่ต้องการก่อนกด key action
- ตัวอย่างการใช้งาน:
  ```java
  // ย้าย cursor ไปที่แถว 10 คอลัมน์ 5
  setCursor(10, 5);

  // กด Enter ที่ตำแหน่งนี้
  execute(null, HostConstants.KEY_ENTER_STR);
  ```
- **ข้อควรระวัง:**  
  - ตรวจสอบตำแหน่ง row/col ให้ตรงกับที่หน้าจอ IBM i ต้องการ  
  - หากไม่ setCursor ไปตำแหน่งที่ถูกต้อง อาจทำให้ action ไม่เกิดผลหรือ flow ผิดพลาด

---

### ตัวอย่างการกรอกข้อมูลคอมเมนต์ในหน้า Work with Order Comments

ตัวอย่างนี้แสดงการกรอกข้อมูลจากรายการคอมเมนต์ในหน้าจอ Work with Order Comments ของ IBM i โดยข้อมูลมาจาก DataList "CusComment"

**รูปแบบข้อมูล Input:**
```json
"CusCommentList": [
  {
    "ID": "e959d7a763f6446a85e9715af0c5052c",
    "OrderNo": "",
    "Comment": "ราคารวมค่าโกดัง ตันละ 600 บาท",
    "FlagInv": "N",
    "FlagPick": "N",
    "FlagAck": "N",
    "FlagBol": "N",
    "FlagPak": "N"
  }
]
```

**ตัวอย่างหน้าจอ IBM i:**
- หน้าจอ "Work with Order Comments" มีช่องคอมเมนต์เริ่มต้นที่แถว 11
- แต่ละแถวมีช่องสำหรับคอมเมนต์ (คอลัมน์ 2) และ Flag ต่างๆ ทางด้านขวา (คอลัมน์ 79)
- แสดงได้ 10 รายการต่อหน้า (ต้องกด Page Down เพื่อเลื่อนไปหน้าถัดไป)

**โค้ดตัวอย่าง:**
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

    logger.info("เริ่มล้างและกรอก CusCommentList ทั้งหมด " + children.size() + " รายการ");
    logger.info("totalPages: " + totalPages);
    
    // กรอกข้อมูลใหม่
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

        logger.info("กรอก Comment row " + currentRow + ": \"" + comment + "\" (FlagInv=" + flagInv + ")");

        currentRow++;
        index++;

        if (index % itemsPerPage == 0 && index < children.size()) {
            execute(null, HostConstants.KEY_PAGEDOWN_STR);
            currentRow = startRow;
        }
    }
    logger.info("บันทึกคอมเมนต์ทั้งหมดเรียบร้อยแล้ว");
} else {
    logger.warn("CusCommentList ไม่มีข้อมูลให้กรอก.");
}
```

**คำอธิบาย:**
- โค้ดนี้อ่านข้อมูลจาก DataList "CusCommentList" และกรอกลงในหน้าจอ Work with Order Comments
- ใช้ getService().setText เพื่อกรอกข้อความและค่า flag
- มีการจัดการเรื่องการแบ่งหน้าโดยการกด Page Down เมื่อกรอกข้อมูลครบหน้า
- มีการบันทึก log ทุกขั้นตอนเพื่อการติดตามและแก้ไขปัญหา

**ข้อควรระวัง:**
- ตรวจสอบให้แน่ใจว่าค่า startRow, commentCol และ flagCol ตรงกับตำแหน่งบนหน้าจอจริง
- ควรตรวจสอบว่าหน้าจอที่กำลังทำงานอยู่เป็นหน้า Work with Order Comments จริงด้วย checkScreen
- ขนาด comment ต้องไม่เกินความยาวที่หน้าจอรองรับ (ควรตัดหรือจัดการถ้าความยาวเกิน)

