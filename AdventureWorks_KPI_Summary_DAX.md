# خلاصه KPI ها و فرمول‌های DAX بر اساس مدل فروش AdventureWorks

این فایل شامل مهم‌ترین KPIهای پیشنهادی بر اساس مدل دیتای شما در Power BI است، به همراه دسته‌بندی و فرمول‌های DAX هر کدام.

---

## 1) KPIهای پایه‌ی فروش و سود (Sales & Profit)

### 1.1. مجموع فروش (Total Sales)

```DAX
Total Sales :=
SUM ( Sales[Sales Amount] )
```

### 1.2. مجموع تعداد فروش (Total Quantity)

```DAX
Total Quantity :=
SUM ( Sales[Order Quantity] )
```

### 1.3. مجموع هزینه محصول (Total Cost)

```DAX
Total Cost :=
SUM ( Sales[Total Product Cost] )
```

### 1.4. سود ناخالص (Gross Profit)

```DAX
Gross Profit :=
[Total Sales] - [Total Cost]
```

### 1.5. درصد حاشیه سود (Gross Margin %)

```DAX
Gross Margin % :=
DIVIDE ( [Gross Profit], [Total Sales] )
```

---

## 2) KPIهای تخفیف (Discount)

فرض بر این است که:
- Sales[Extended Amount] = مبلغ قبل از تخفیف
- Sales[Sales Amount] = مبلغ بعد از تخفیف

### 2.1. مبلغ تخفیف کل (Discount Amount)

```DAX
Discount Amount :=
SUM ( Sales[Extended Amount] ) - SUM ( Sales[Sales Amount] )
```

### 2.2. درصد تخفیف نسبت به مبلغ اولیه (Discount %)

```DAX
Discount % :=
DIVIDE ( [Discount Amount], SUM ( Sales[Extended Amount] ) )
```

---

## 3) KPIهای کانال فروش (Sales Channel)

از جدول 'Sales Order' ستون Channel استفاده می‌کنیم (مثلاً "Internet" و "Reseller").

### 3.1. فروش اینترنت (Internet Sales)

```DAX
Internet Sales :=
CALCULATE (
    [Total Sales],
    'Sales Order'[Channel] = "Internet"
)
```

### 3.2. فروش نماینده (Reseller Sales)

```DAX
Reseller Sales :=
CALCULATE (
    [Total Sales],
    'Sales Order'[Channel] = "Reseller"
)
```

### 3.3. سهم هر کانال از کل فروش (Channel Mix %)

```DAX
Internet Sales % :=
DIVIDE ( [Internet Sales], [Total Sales] )

Reseller Sales % :=
DIVIDE ( [Reseller Sales], [Total Sales] )
```

---

## 4) KPIهای سفارش (Order-Level KPIs)

از جدول 'Sales Order' برای سطح سفارش استفاده می‌کنیم.

### 4.1. تعداد سفارش‌ها (Order Count)

```DAX
Order Count :=
DISTINCTCOUNT ( 'Sales Order'[Sales Order] )
```

### 4.2. میانگین مبلغ هر سفارش (Average Order Value - AOV)

```DAX
Average Order Value :=
DIVIDE ( [Total Sales], [Order Count] )
```

---

## 5) KPIهای زمانی (Time Intelligence)

با فرض وجود جدول Date و ستون Date و رابطه فعال بین Sales و Date (مثلاً از طریق OrderDateKey).

### 5.1. فروش سال‌جاری (Year-to-Date Sales - YTD)

```DAX
Sales YTD :=
TOTALYTD (
    [Total Sales],
    'Date'[Date]
)
```

### 5.2. فروش سال قبل (Last Year Sales)

```DAX
Total Sales LY :=
CALCULATE (
    [Total Sales],
    DATEADD ( 'Date'[Date], -1, YEAR )
)
```

### 5.3. رشد فروش نسبت به سال قبل (Sales YoY %)

```DAX
Sales YoY % :=
VAR CurrentSales  = [Total Sales]
VAR LastYearSales = [Total Sales LY]
RETURN
    DIVIDE ( CurrentSales - LastYearSales, LastYearSales )
```

### 5.4. فروش ۱۲ ماه اخیر (Rolling 12 Months)

```DAX
Sales Rolling 12M :=
CALCULATE (
    [Total Sales],
    DATESINPERIOD (
        'Date'[Date],
        MAX ( 'Date'[Date] ),
        -12,
        MONTH
    )
)
```

---

## 6) KPIهای مشتری و فروشنده (Customer & Reseller KPIs)

### 6.1. تعداد مشتریان یکتا (Customer Count)

```DAX
Customer Count :=
DISTINCTCOUNT ( Customer[CustomerKey] )
```

### 6.2. فروش به ازای هر مشتری (Sales per Customer)

```DAX
Sales per Customer :=
DIVIDE ( [Total Sales], [Customer Count] )
```

### 6.3. تعداد Resellerها (Reseller Count)

```DAX
Reseller Count :=
DISTINCTCOUNT ( Reseller[ResellerKey] )
```

### 6.4. فروش به ازای هر Reseller

```DAX
Sales per Reseller :=
DIVIDE ( [Total Sales], [Reseller Count] )
```

---

## 7) KPIهای محصول و دسته‌بندی (Product & Category KPIs)

با استفاده از جدول Product (حاوی ستون‌هایی مثل Category ،Subcategory ،Product و ...).

### 7.1. فروش محصول (Product Sales)

در ویژوال‌هایی که Product روی محور قرار دارد، همین Measure پایه کافی است:

```DAX
Product Sales :=
[Total Sales]
```

(در هر ردیف محصول، کانتکست فیلتر باعث می‌شود فقط فروش همان محصول محاسبه شود.)

### 7.2. فروش دسته دوچرخه (Bike Sales)

```DAX
Bike Sales :=
CALCULATE (
    [Total Sales],
    Product[Category] = "Bikes"
)
```

### 7.3. سهم Bikes از کل فروش (Bike Sales %)

```DAX
Bike Sales % :=
DIVIDE ( [Bike Sales], [Total Sales] )
```

---

## 8) نکته مهم در مورد ستون Product[totalSales]

به جای ساختن ستون محاسباتی مانند:

```DAX
Product[totalSales] =
CALCULATE ( SUM ( Sales[Sales Amount] ) )
```

بهتر است از Measure زیر استفاده شود که داینامیک است و به فیلترها پاسخ می‌دهد:

```DAX
Total Sales :=
SUM ( Sales[Sales Amount] )
```

و این Measure را در هر ویژوال (مثلاً بر اساس Product، Customer، Territory و ...) استفاده کنید.

---

### نکات نهایی

- همه KPIها را می‌توانی در Power BI داخل یک Display Folder به نام **KPIs** یا دسته‌بندی‌های مشابه (Sales, Margin, Customer, Product, Time) مرتب کنی.
- برای جلوگیری از خطاهای تقسیم بر صفر، همیشه از تابع `DIVIDE` استفاده شده است.
- در صورت تغییر نام جدول‌ها/ستون‌ها در مدل خودت، فقط نام آن‌ها را در فرمول‌ها به‌روزرسانی کن.

