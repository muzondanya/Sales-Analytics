# Sales-Analytics
1. Category Revenue Percentage by Continent
Підраховується загальний дохід, дохід від категорії "Bookcases & shelving units", відсоток доходу від категорії "Bookcases & shelving units" від загального доходу по континенту
SELECT
     sp.continent,
     sum(p.price) as revenue,
     sum(case when p.category = 'Bookcases & shelving units' then  p.price end ) as revenue_from_bookcases,
     sum(case when p.category = 'Bookcases & shelving units' then  p.price end ) / sum(p.price) * 100 as revenue_from_bookcases_percent
FROM `data-analytics-mate.DA.order` o
join `data-analytics-mate.DA.product` p
on o.item_id = p.item_id
join `data-analytics-mate.DA.session_params` sp
on o.ga_session_id = sp.ga_session_id
group by sp.continent
order by 2 desc
2. User engagement events
Прораховується к-сть івентів, що мають тип user_engagement, але лише по тих сесіях, де в цілому було більше 2 будь-яких івентів в рамках сесії
SELECT
 COUNT(*)AS engagement_event
FROM
 `data-analytics-mate.DA.event_params` sp
JOIN (
 SELECT
   ga_session_id,
   COUNT(*) AS event_cnt
 FROM
   `data-analytics-mate.DA.event_params`
 GROUP BY
   ga_session_id
 HAVING
   COUNT(*) > 2) active_session
ON
 sp.ga_session_id = active_session.ga_session_id
WHERE
 ep.event_name = 'user_engagement'
3. Prediction fulfillment
Обчислюється відсоток виконання накопичувальних доходів від накопичувальних цілей (predict) в розрізі дня.
Select
   date,
   sum(revenue) over(order by date) as running_revenue,
   sum(predict) over(order by date) as running_predict,
   sum(revenue) over(order by date) /
   sum(predict) over(order by date) * 100 as goal_percent
from (
Select
   date,
   sum(predict) as predict,
   sum(revenue) as revenue
from (
SELECT
   date,
   predict,
   0 as revenue
FROM `data-analytics-mate.DA.revenue_predict`
union all
Select
   s.date,
   0 as predict,
   sum(p.price) as revenue
from `DA.order` o
join `DA.product` p
on o.item_id = p.item_id
join `DA.session` s
on o.ga_session_id = s.ga_session_id
group by s.date)union_data
group by date
) date_data
4. Specify the language
Вибираються сесії, де поле language містить англійську мову з уточненнями (наприклад, en-us, en-gb).
Згруповані результати за цими уточненнями та підрахована кількість сесій для кожного уточнення.
SELECT
 SUBSTR(LANGUAGE, STRPOS(LANGUAGE, '-')+1, 2) AS en_type,
 COUNT(*) AS session_cnt
FROM
 `data-analytics-mate.DA.session_params`
WHERE
 LANGUAGE LIKE 'en-%'
GROUP BY
 1
ORDER BY
 2 DESC
