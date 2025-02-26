# Python
1. Top 3 product_ids with the highest volume.
```python
# Top 3 product_id có volume cao nhất
top_3_product_ids = payment_enriched.groupby('product_id')['volume'].sum().nlargest(3)
print(top_3_product_ids)
```
*Product_id của Top 3 sản phẩm có hiệu suất lớn nhất: 1976, 429, 372*
**2**. **Given that 1 product_id is only owed by 1 team, are there any abnormal products against this rule?**
```python
# Kiểm tra sản phẩm thuộc nhiều hơn 1 team
team_check = payment_enriched.groupby('product_id')['team_own'].nunique()
abnormal_products = team_check[team_check > 1]
print(abnormal_products)
```
*=> Không có sản phẩm nào thuộc nhiều hơn 1 team*
**3. Find the team has had the lowest performance (lowest volume) since Q2.2023. Find the category that contributes the least to that team.**
```python
# Lọc các dữ liệu từ Q2.2023 trở đi
payment_enriched['report_month'] = pd.to_datetime(payment_enriched['report_month'], format='%Y-%m')
q2_2023 = payment_enriched[payment_enriched['report_month'] >= '2023-04']

# Team có hiệu suất thấp nhất
team_lowest_performance = q2_2023.groupby('team_own')['volume'].sum().nsmallest(1)
print(team_lowest_performance)

# Tìm category đóng góp ít nhất trong team đó
lowest_team = team_lowest_performance.idxmin()
lowest_team_category = q2_2023[q2_2023['team_own'] == lowest_team].groupby('category')['volume'].sum().nsmallest(1)
print(lowest_team_category)
```
*- Team có hiệu suất thấp nhất: APS (volume: 51141753)*
*- Category đóng góp ít nhất: PXXXXXE (volume: 25232438)*
**4. Find the contribution of source_ids of refund transactions (payment_group = ‘refund’), what is the source_id with the highest contribution?**
```python
# Tìm source_id có đóng góp cao nhất cho giao dịch 'refund'
refunds = payment_enriched[payment_enriched['payment_group'] == 'refund']
top_refund_source = refunds.groupby('source_id')['volume'].sum().idxmax()
print(top_refund_source)
```
*Source_id có bị hoàn tiền nhiều nhất là 38*
**5. Define type of transactions (‘transaction_type’) for each row**
```python
# Thiết lập hàm để xác định các loại giao dịch
def classify_transaction(row):
    if row['transType'] == 2 and row['merchant_id'] == 1205:
        return 'Bank Transfer Transaction'
    elif row['transType'] == 2 and row['merchant_id'] == 2260:
        return 'Withdraw Money Transaction'
    elif row['transType'] == 2 and row['merchant_id'] == 2270:
        return 'Top Up Money Transaction'
    elif row['transType'] == 2:
        return 'Payment Transaction'
    elif row['transType'] == 8 and row['merchant_id'] == 2250:
        return 'Transfer Money Transaction'
    elif row['transType'] == 8:
        return 'Split Bill Transaction'
    else:
        return 'Invalid Transaction'

# Áp dụng hàm để thêm cột 'transaction_type'
transactions['transaction_type'] = transactions.apply(classify_transaction, axis=1)

# Kiểm tra kết quả
transactions[['transType', 'merchant_id', 'transaction_type']].head()
```
![image](https://github.com/user-attachments/assets/2c153c4d-4ee2-441c-b263-576613406b5d)

**6. Of each transaction type (excluding invalid transactions): find the number of transactions, volume, senders and receivers.**
```python
import pandas as pd

# Giả sử transactions là DataFrame chứa dữ liệu giao dịch
summary_stats = transactions.groupby('transaction_type').agg(
    number_of_transactions=('transaction_type', 'count'),
    total_volume=('volume', 'sum'),
    unique_senders=('sender_id', 'nunique'),
    unique_receivers=('receiver_id', 'nunique')
).reset_index()

# Kiểm tra kết quả
print(summary_stats)
```
