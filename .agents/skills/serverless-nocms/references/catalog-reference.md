# Product catalog reference

Use this only when the user wants to reproduce or adapt the David Meatball catalog. Names and Myanmar labels are copied as text from the reference project; preserve spelling rather than auto-translating. Confirm that these products and packaging details still apply before seeding a new instance. Do not seed report history or assume old prices are current.

## Active food catalog baseline

For skewered food products, the usual stock unit is `ไม้`, purchase unit is `ถุง`, and the table's last two columns are recommended bags and items per bag. The optional Myanmar labels are reference text and must be reviewed with the user before changing.

| Source # | Thai name | Myanmar label | Bags | Items/bag |
| ---: | --- | --- | ---: | ---: |
| 1 | พรชัยลูกชิ้นกลาง | မှိုထမဒါအိတ်အနီ | 2 | 32 |
| 2 | เม็ดเบทาโกไส้กรอก | ဆိုဒ်ကောဒယ် | 4 | 28 |
| 3 | ไก่ยอพริก | အစပ်အပြားအနီ၃ပြား | 2 | 28 |
| 4 | นมวานิลา (ไส้กรอกนม) | အပန်းရောင်စံပယ်အနံ့ | 2 | 21 |
| 5 | จัมโบ้ทอง (ไส้กรอกใหญ่) | ဆီုဒ်ကောယိုင်၁၀ဘတ်တန် | 1 | 15 |
| 6 | หมูอร่อย | မှိုမိတ်ချိ | 2 | 22 |
| 7 | ลูกชิ้นแฟนซีจัมโบ้ | ကိုင်၃လုံး(၁၀ဘတ်တန်) | 1 | 19 |
| 8 | เต้าหู้ปลากุนซือ | တဟူးလေးထောင့် | 2 | 17 |
| 9 | ไก่อร่อย (ไส้กรอกไก่หนังกรอบ) | ကိုင်လောင်ကော် | 2 | 22 |
| 10 | ชีสเขียว | ချိခရုံ | 2 | 20 |
| 12 | ลูกชิ้นหมึก (ลูกชิ้นปลาหมึกผสมไก่) | လုချင်းပါမူး | 2 | 15 |
| 13 | จ๊อปู | ကျော့ပူး | 1 | 30 |
| 14 | เมก้าแคลป (ปูอัด) | ပူးအား(10ဘတ်) | 1 | 12 |
| 15 | จ๊อปูอัด | ကျော့ပူအာ | 1 | 36 |
| 16 | โบราณ (ไก่กรอกแฟรงก์รมควันชีส) | ချိလောင်ကော် | 1 | 28 |
| 17 | เจเจ (เต้าหู้หมู) | တဟူးမှို | 2 | 19 |
| 18 | จัมโบ้ทองชีส | ကျန်ဘို့ချိစ်(10ဘတ်) | 1 | 18 |
| 19 | ปลาทะเลแหลม | ပါလန်လုချင်းပါးအဖြူ | 3 | 18 |
| 20 | พรชัยเอ็นลูกเล็ก | အစ်မှိုအိတ်အပြာ | 2 | 29 |
| 21 | ไก่พันสาหร่าย | ဖန်စလိုင် | 3 | 14 |
| 22 | ปูอัดคนิจังสีเขียว | ပူအားလစ် | 1 | 28 |

## Other active items

| Source # | Thai name | Myanmar label | Stock unit | Purchase unit | Behavior |
| ---: | --- | --- | --- | --- | --- |
| 23 | เกี๊ยว | ကီယုံး(10ဘတ်) | ถุง | ถุง | Has a sale price and purchase cost; use a current, user-confirmed margin/price. Historical reference target was about 30% gross margin. |
| 26 | น้ำจิ้มจอมยุทธ | အချဥ်ရည် | กล่อง | กล่อง | Complimentary with food; no standalone sale price; track purchase cost when ordered (`ORDER_COST_ONLY`). |

The catalog may contain additional non-stock rows or legacy-only products. If needed for old imported reports, retain them as inactive with the exact historical names and units so old snapshots remain readable; do not show them in new public submissions.

## Pricing and history rules

- Sale prices are per stock unit; purchase prices are per purchase unit. Store the source/effective date for purchase-price changes.
- Treat price values in old migrations or sheets as dated sample inputs, not current truth. Obtain owner-confirmed current prices before a new production seed.
- Snapshot product names, labels, units, conversion metadata, sale/purchase prices, price date, and cost mode into each report line. A later price edit affects new reports only.
- For product 26, keep sale price null and count purchase cost when the order quantity is recorded. Do not show it as an individually sold item.
