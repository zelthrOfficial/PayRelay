# PayRelay REST API ฟรี | PromptPay QR, ตรวจสอบสลิป และ TrueMoney

<p align="center">
  <img src="https://payrelay.zelthr.rest/favicon.png" width="144" alt="PayRelay logo" />
</p>

PayRelay คือ REST API ฟรีสำหรับนักพัฒนาและร้านค้าไทย ใช้สร้าง PromptPay QR ตรวจสอบสลิป และตรวจหรือแลกซองของขวัญ TrueMoney ได้ใน API เดียว

- API: <https://payrelay.zelthr.rest>
- คู่มือแบบละเอียด: <https://payrelay.zelthr.rest/docs>
- ข้อกำหนด: <https://payrelay.zelthr.rest/docs/terms>
- ความเป็นส่วนตัว: <https://payrelay.zelthr.rest/docs/privacy>

## เริ่มใช้ PayRelay API ฟรี

ไม่ต้องสมัครบัญชีหรือใช้ API key ส่ง request เป็น JSON โดยระบุ `Content-Type: application/json`

```bash
curl https://payrelay.zelthr.rest/healthz
```

ถ้า API พร้อมใช้งาน จะได้รับ HTTP `200` และ `"success": true`

## สร้าง PromptPay QR ฟรีด้วย API

ส่งเบอร์ PromptPay เลขประจำตัวประชาชน หรือ e-Wallet ID พร้อมยอดเงินบาท ระบบรับจำนวนเงินเป็น string หรือ number แนะนำให้ใช้ string เมื่อมียอดทศนิยม

```bash
curl -X POST https://payrelay.zelthr.rest/v1/slips/qrcode \
  -H 'Content-Type: application/json' \
  -d '{"account":"0812345678","amount":"100.50"}'
```

รูป QR ที่ได้อยู่ใน `data.img` ในรูปแบบ `data:image/png;base64,...` นำค่านี้ไปแสดงเป็นรูปภาพได้เลย

## API ตรวจสอบสลิปฟรี

ส่งข้อมูล QR จากสลิปและระบุยอดเงินที่ต้องการตรวจ

```bash
curl -X POST https://payrelay.zelthr.rest/v1/slips/verify \
  -H 'Content-Type: application/json' \
  -d '{"qrcode":"<ข้อมูล QR จากสลิป>","amount":"100.50"}'
```

เมื่อสำเร็จ `data` จะมีวันเวลาทำรายการ ยอดเงิน ผู้โอน ผู้รับ และเลขอ้างอิงรายการ

## API ตรวจสอบซอง TrueMoney ฟรี

ใช้ endpoint นี้ตรวจสอบซองของขวัญ TrueMoney ก่อนแลก ระบบจะไม่แลกซองให้

```bash
curl -X POST https://payrelay.zelthr.rest/v1/tmn/verify \
  -H 'Content-Type: application/json' \
  -d '{"gift":"<ลิงก์หรือรหัสซอง>","amount":"80.00"}'
```

`amount` เป็นตัวเลือก หากส่งมา ระบบจะตรวจว่ายอดตรงกับซอง

## API แลกซองของขวัญ TrueMoney

ตรวจลิงก์ซอง เบอร์โทรศัพท์ผู้รับ และจำนวนเงินให้ถูกต้องก่อนส่งคำขอ การแลกซองเป็นรายการจริง

```bash
curl -X POST https://payrelay.zelthr.rest/v1/tmn/redeem \
  -H 'Content-Type: application/json' \
  -d '{"gift":"<ลิงก์หรือรหัสซอง>","phone":"0812345678","amount":"80.00"}'
```

`phone` คือเบอร์มือถือไทยของผู้รับ ส่วน `amount` เป็นตัวเลือกสำหรับตรวจยอดก่อนแลก เมื่อรายการสำเร็จ `data` จะมีวันเวลา ยอดเงิน และเลขอ้างอิงรายการ

## PayRelay API ฟรีไหม?

PayRelay เปิดให้เรียกใช้ API ฟรี ไม่ต้องสมัครสมาชิกและไม่ต้องใช้ API key ทั้งนี้แต่ละ endpoint มี rate limit เพื่อควบคุมจำนวนคำขอ ดูวิธีใช้และตัวอย่าง request เพิ่มเติมได้ที่ [เอกสาร PayRelay REST API](https://payrelay.zelthr.rest/docs)

## รูปแบบผลลัพธ์

ทุก endpoint ส่งผลลัพธ์ในรูปแบบเดียวกัน

```json
{
  "success": true,
  "data": {}
}
```

เมื่อเกิดข้อผิดพลาด API จะส่ง `success: false` และรายละเอียดใน `error`

```json
{
  "success": false,
  "error": {
    "code": "INVALID_AMOUNT",
    "message": "จำนวนเงินไม่ถูกต้อง",
    "details": {
      "field": "amount"
    }
  }
}
```

ตรวจ HTTP status และ `error.code` ก่อนจัดการผลลัพธ์ เช่น `400` หรือ `422` สำหรับข้อมูลที่ไม่ถูกต้อง, `409` สำหรับรายการซ้ำ, `429` เมื่อส่งคำขอเกินกำหนด, `502` เมื่อยืนยันรายการไม่สำเร็จ และ `504` เมื่อคำขอหมดเวลา การแลกซองที่ได้ผลลัพธ์ไม่ชัดเจนไม่ควรส่งซ้ำทันที ให้ตรวจสอบรายการก่อน

ข้อกำหนด request:

- ส่ง body เป็น JSON object ขนาดไม่เกิน 100 KB
- จำนวนเงินเป็นบาท รองรับทศนิยมไม่เกิน 2 ตำแหน่ง
- จำกัดค่าเริ่มต้น 30 คำขอต่อนาทีต่อ IP หากได้ `429` ให้รอตาม `Retry-After`
- ทุก response มี `X-Request-ID` สำหรับอ้างอิงเมื่อขอความช่วยเหลือ

## สถิติ API

```bash
curl https://payrelay.zelthr.rest/v1/stats
```

แสดงจำนวนคำขอ ยอดรวมของสลิปที่ตรวจสำเร็จกับซองที่แลกสำเร็จ จำนวนสลิปที่ตรวจสำเร็จ และจำนวนซองที่แลกสำเร็จ ยอดรวมส่งกลับเป็น string หน่วย THB

ดู request และ response ของทุก endpoint ได้ที่ [เอกสาร API](https://payrelay.zelthr.rest/docs) รวมถึง [รหัสข้อผิดพลาด](https://payrelay.zelthr.rest/docs/errors)

## ติดต่อ

- [Discord PayRelay](https://discord.gg/ba48QSbYh8)
- [Discord ผู้พัฒนา](https://discord.gg/UQuwAgSvmG)
