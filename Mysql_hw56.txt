USE cc;

SELECT @@sql_mode;
SET sql_mode = (SELECT CONCAT(@@SQL_MODE, ",ONLY_FULL_GROUP_BY"));
SET sql_mode = (SELECT REPLACE(@@SQL_MODE, "ONLY_FULL_GROUP_BY", ""));

-- Q1
CREATE OR REPLACE VIEW revenue AS
SELECT memberships.Member_Number, memberships.Membership_Type, memberships.Family_Name, memberships.Year_Joined,
		Golf_payment, Tennis_payment, Pool_payment, Dining_payment, Other_payment,
        CASE WHEN Promo_1_date IS NOT NULL THEN 1 ELSE 0 END AS Promo_1,
        CASE WHEN promotwo.Member_Number IS NOT NULL THEN 1 ELSE 0 END AS Promo_2
FROM memberships
	LEFT JOIN ( SELECT golf.Member_Number, SUM(Amount) AS Golf_payment
				FROM golf
                #WHERE Description = "Golf_Lessons"
				GROUP BY golf.Member_Number
				) AS golf_lesson
		ON golf_lesson.Member_Number = memberships.Member_Number
	LEFT JOIN ( SELECT tennis.Member_Number, SUM(Amount) AS Tennis_payment
				FROM tennis
				GROUP BY tennis.Member_Number
				) AS tennis_play
		ON tennis_play.Member_Number = memberships.Member_Number
	LEFT JOIN ( SELECT pool.Member_Number, SUM(Amount) AS Pool_payment
				FROM pool
				GROUP BY pool.Member_Number
				) AS pool
		ON pool.Member_Number = memberships.Member_Number 
	LEFT JOIN ( SELECT dining.Member_Number, SUM(Total) AS Dining_payment
				FROM dining
				GROUP BY dining.Member_Number
				) AS dining
		ON dining.Member_Number = memberships.Member_Number 
	LEFT JOIN ( SELECT other.Member_Number, SUM(Amount) AS Other_payment
				FROM other
				GROUP BY other.Member_Number
				) AS other
		ON other.Member_Number = memberships.Member_Number 
	LEFT JOIN (SELECT promoone.Member_Number, Date AS Promo_1_date 
    FROM promoone) AS pone ON pone.Member_Number = memberships.Member_Number
	LEFT JOIN promotwo ON promotwo.Member_Number = memberships.Member_Number
ORDER BY memberships.Member_Number;
        
SELECT *
FROM revenue;
#LIMIT 25;

CREATE OR REPLACE VIEW members_activities AS
SELECT memberships.Member_Number, memberships.Membership_Type, memberships.Family_Name, memberships.Year_Joined,
		Thanksgiving, `Easter Brunch`, `4th of July`, `Private Function`,
        CASE WHEN Promo_1_date IS NOT NULL THEN 1 ELSE 0 END AS Promo_1,
        CASE WHEN promotwo.Member_Number IS NOT NULL THEN 1 ELSE 0 END AS Promo_2
FROM memberships
LEFT JOIN ( SELECT special.Member_Number, special.Thanksgiving
				FROM special
				GROUP BY special.Member_Number
				) AS Thanksgiving
		ON Thanksgiving.Member_Number = memberships.Member_Number
LEFT JOIN ( SELECT special.Member_Number, special.`Easter Brunch`
				FROM special
				GROUP BY special.Member_Number
				) AS EB
		ON EB.Member_Number = memberships.Member_Number
LEFT JOIN ( SELECT special.Member_Number, special.`4th of July`
				FROM special
				GROUP BY special.Member_Number
				) AS 4th_of_July
		ON 4th_of_July.Member_Number = memberships.Member_Number
LEFT JOIN ( SELECT special.Member_Number, special.`Private Function`
				FROM special
				GROUP BY special.Member_Number
				) AS PF
		ON PF.Member_Number = memberships.Member_Number
LEFT JOIN (SELECT promoone.Member_Number, Date AS Promo_1_date 
    FROM promoone) AS pone ON pone.Member_Number = memberships.Member_Number
	LEFT JOIN promotwo ON promotwo.Member_Number = memberships.Member_Number
ORDER BY memberships.Member_Number;

SELECT *
FROM members_activities;
#LIMIT 25;

CREATE OR REPLACE VIEW golf_payment AS
SELECT memberships.Member_Number, Membership_Type, Family_Name, Lesson_payment, Green_Fee, Golf_Shop
FROM memberships
LEFT JOIN ( SELECT golf.Member_Number, SUM(Amount) AS Lesson_payment
				FROM golf
                WHERE Description = "Golf_Lessons"
				GROUP BY golf.Member_Number
				) AS golf_lesson
		ON golf_lesson.Member_Number = memberships.Member_Number
LEFT JOIN ( SELECT golf.Member_Number, SUM(Amount) AS Green_Fee
				FROM golf
                WHERE Description = "Green_Fee"
				GROUP BY golf.Member_Number
				) AS golf_green
		ON golf_green.Member_Number = memberships.Member_Number
LEFT JOIN ( SELECT golf.Member_Number, SUM(Amount) AS Golf_Shop
				FROM golf
                WHERE Description = "Golf_Shop"
				GROUP BY golf.Member_Number
				) AS golf_shop
		ON golf_shop.Member_Number = memberships.Member_Number
ORDER BY memberships.Member_Number;

SELECT *
FROM golf_payment;
#LIMIT 25;

-- Q2
SELECT SUM(Lesson_payment) AS Total_Lesson_payment, 
SUM(Green_Fee) AS Total_Green_Fee, SUM(Golf_Shop) AS Total_Golf_Shop
FROM golf_payment;

SELECT SUM(Golf_payment) AS Golf_Rev, 
	SUM(Tennis_payment) AS Tennis_Rev, 
    SUM(Pool_payment) AS Pool_Rev, 
    SUM(Dining_payment) AS Dining_Rev, 
    SUM(Other_payment) AS Other_Rev
FROM revenue;

-- Q4
SELECT Membership_Type, COUNT(Membership_Type) AS Number_type
FROM revenue
WHERE Dining_payment IS NOT NULL 
	AND Golf_payment IS NULL
	AND Tennis_payment IS NULL
    AND Other_payment IS NULL
    AND Pool_payment IS NULL
GROUP BY Membership_Type;

-- Q5
SELECT 
    Membership_Type,
    SUM(Promo_1) AS Promo1_Count,
    SUM(Promo_2) AS Promo2_Count,
    COUNT(*) AS Total_Members
FROM members_activities
GROUP BY Membership_Type;

SELECT 
    CASE 
        WHEN Promo_1 = 1 AND Promo_2 = 1 THEN 'Both'
        WHEN Promo_1 = 1 THEN 'Promo1 Only'
        WHEN Promo_2 = 1 THEN 'Promo2 Only'
        ELSE 'Neither'
    END AS Promo_Participation,
    COUNT(*) AS Member_Count,
    AVG(Dining_payment) AS Avg_Dining,
    AVG(Golf_payment) AS Avg_Golf,
    AVG(Tennis_payment) AS Avg_Tennis,
    AVG(Pool_payment) AS Avg_Pool,
    AVG(Other_payment) AS Avg_Other
FROM revenue
GROUP BY Promo_Participation;
