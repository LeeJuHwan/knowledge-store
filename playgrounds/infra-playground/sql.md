---
description: 조회 성능 개선하기 미션을 진행할 때 최소한으로 필요한 SQL 역량이 있는지를 확인하기 👍
---

# SQL, 이 정도는 알아야지 😎

<details>

<summary>만약 SQL 리부트를 하고 싶다면?</summary>

[https://www.youtube.com/watch?v=\_DgpFbcGuAc](https://www.youtube.com/watch?v=_DgpFbcGuAc)

</details>



### 실습하기

{% hint style="info" %}
[해당 사이트](https://www.w3schools.com/sql/trymysql.asp?filename=trysql_func_mysql_concat)에서 아래 쿼리를 작성해보세요.&#x20;
{% endhint %}

{% stepper %}
{% step %}
### 200개 이상 팔린 상품명과 그 수량을 수량 기준 내림차순으로 보여주세요.

```sql
SELECT 
	pd.ProductID AS "상품아이디",
	pd.ProductName AS "상품명",
	SUM(od.Quantity) AS "총수량"
FROM OrderDetails AS od
INNER JOIN Products AS pd
	ON od.ProductID = pd.ProductID
GROUP BY pd.ProductID
HAVING SUM(od.Quantity) >= 200
ORDER BY SUM(od.Quantity) DESC;
```

<figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### 많이 주문한 순으로 고객 리스트(ID, 고객명)를 구해주세요. (고객별 구매한 물품 총 갯수)

```sql
SELECT
    cs.CustomerID AS "고객아이디",
    cs.CustomerName AS "고객이름",
    SUM(Quantity) AS "주문량"
FROM Orders AS od
INNER JOIN Customers AS cs
	ON cs.CustomerID = od.CustomerID
INNER JOIN OrderDetails AS od_detail
	ON od.OrderID = od_detail.OrderID
GROUP BY cs.CustomerID
ORDER BY SUM(Quantity) DESC;
```

<figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### 많은 돈을 지출한 순으로 고객 리스트를 구해주세요.

```sql
SELECT
    cs.CustomerID AS "고객아이디",
    cs.CustomerName AS "고객이름",
    SUM(Quantity * Price) AS "지출금액($)"
FROM Orders AS od
INNER JOIN Customers AS cs
	ON cs.CustomerID = od.CustomerID
INNER JOIN OrderDetails AS od_detail
	ON od.OrderID = od_detail.OrderID
INNER JOIN Products AS pd
	ON od_detail.ProductID = pd.ProductID
GROUP BY cs.CustomerID
ORDER BY SUM(Quantity * Price) DESC;
```

<figure><img src="../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>
{% endstep %}
{% endstepper %}



