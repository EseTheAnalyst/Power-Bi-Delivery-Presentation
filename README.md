# Power-Bi-Delivery-Presentation
The objective of this presentation is to show  the delivery partners across three key performance indicators and demonstrate the strongest overall service quality.

A single-page Power BI dashboard that compares delivery partners on delivered, delayed and failed rates, performance in rainy weather, and average customer rating.

<img width="2266" height="1288" alt="screenshot" src="https://github.com/user-attachments/assets/97362e29-d350-4761-99cd-1daec8afbf29" />




**Table of contents**

1.	Project overview
2.	Data model
3.	Dashboard layout
4.	Build steps
5.	DAX measures
6.	Troubleshooting log
7.	Key insights


**Project Overview**

Goal: show which delivery partner performs best overall and in rainy conditions, and how customers rate each partner.
Questions the dashboard answers

•	Which partner has the highest delivered rate overall and on rainy days?

•	Which partner has the lowest delayed and failed rates?

•	How does each partner rate with customers?

•	How much do delivery rates drop in the rain?

•	Recommended delivery partner to use

**Tool**: Microsoft Power BI Desktop (Power Query + DAX)

**Data model**


| Item | Details |
|--------|-----------------|
| Main table | Delivery_Logistics table |
| Key columns	 | delivery_partner, delivery_status (delivered / delayed / failed), weather column (rainy vs other), customer rating |
| Partners | Amazon Logistics, Blue Dart, Delhivery, DHL, Ecom Express, Ekart, FedEx, Shadowfax, XpressBees |



**Dashboard layout**


| Area | Content |
|--------|-----------------|
| Header | Title text box and subtitle |
| Row 1	 | Six KPI cards: 3 overall, 3 rainy-day |
| Row 2 | Overall delivery performance chart and rainy-day delivery performance chart |
| Row 3 | Average customer rating chart and Key Business Note |



**Build steps**

1. Plan the layout
•	Used a 16:9 page with a header, KPI cards, two comparison charts and a bottom row for ratings and insights.
•	Gave matching visuals identical sizes and aligned them using Format → Align and Format → General → Properties (exact position and size).


2. **Add a title and subtitle**
   
o	Add a title and subtitle - Inserted a **Rectangle shape** (Insert → Shapes → Rectangle) across the top of the page to act as the header banner. - Typed the title and subtitle directly inside the shape: 
o	Title: Delivery Partner Performance Dashboard
o	Subtitle: Performance - Efficiency - Customer Satisfaction
Aligned the text inside the shape (Format → Shape → Text) so the title and subtitle line up with the visuals below. - Used a rectangle instead of a text box because the text box showed blue spell-check underlines under the title and subtitle. The shape has no such marks.

3. **Rename the visuals**
   
| Visual | Title | Subtitle |
|--------|-----------------|-----------------|
| Overall chart | Overall Delivery Performance by Partner (%) | Delivered, delayed and failed rates, all weather conditions |
| Rainy chart | Rainy-Day Delivery Performance by Partner (%) | Delivered, delayed and failed rates on rainy days only |
| Rating chart | Average Customer Rating by Delivery Partner | Average rating out of 5, axis start at 3.50 to show small difference |

Set under Format → Visual → General → Title.

4. **Rename the KPI cards**
   
| Card | New label |
|--------|-----------------|
| Delhivery 75.2%	| Best Overall Delivered Rate |
| Delhivery 19.8% | Lowest Overall Delayed Rate |
| Ecom Express 4.4% |	Lowest Overall Failed Rate |
| Delhivery 65.7% |	Best Rain Delivered Rate |
| Delhivery 27.7% |	Lowest Rain Delayed Rate |
| Amazon Logistics 5.0% | Lowest Rain Failed Rate |

Colors follow best / middle / worst: green, yellow, red.

5. **Clean the partner names**
   
•	Fixed lowercase and misspelled names (for example delhivery) in Power Query using Transform → Format → Capitalize Each Word and Replace Values, so every visual shows the corrected names.

6. **Build the status charts (delivered, delayed, failed)**
   
•	Converted both delivery charts to stacked bar charts with one measure per status (see DAX measures).
•	Y-axis: delivery_partner.
•	X-axis: Delivered %, then Delayed %, then Failed %, in that order. The drop order sets the segment order.
•	Colors: Delivered = green, Delayed = amber, Failed = red.
•	Turned on data labels (percentage format, 1 decimal place) and sorted partners by delivered rate, highest first.
•	Built the rainy chart the same way using the rainy measures.

7. **Add the Key Business Insight**
   
•	Created a measure that holds the note text (see DAX).
•	Displayed it in a Text Box because line breaks created with UNICHAR(10) did not display in a standard card.
•	Shortened each point so nothing was cut off and fixed the box size.


8. **Add the Recommended Delivery Partner card**
   
•	Added a Recommended Delivery Partner card showing the partner with the highest delivered rate.
•	Title: **Top Recommended Delivery Partner**.

9. **Rating chart axis note**
    
•	The axis started at 3.50, which exaggerates small differences (3.61 vs 3.70).
•	Added a subtitle note: "Average rating out of 5. Axis starts at 3.50 to show small differences."

10. **Final polish**
    
•	Same font and spacing across all visuals, light gray page background, white visual backgrounds.
•	Data labels on every chart.

**DAX measures**

**Status percentages (Overall)**

Delivered % =
DIVIDE(
    CALCULATE(COUNTROWS('Delivery_Logistics table'),
        'Delivery_Logistics table'[delivery_status] = "delivered"),
    COUNTROWS('Delivery_Logistics table')
)

Delayed % =
DIVIDE(
    CALCULATE(COUNTROWS('Delivery_Logistics table'),
        'Delivery_Logistics table'[delivery_status] = "delayed"),
    COUNTROWS('Delivery_Logistics table')
)

Failed % =
DIVIDE(
    CALCULATE(COUNTROWS('Delivery_Logistics table'),
        'Delivery_Logistics table'[delivery_status] = "failed"),
    COUNTROWS('Delivery_Logistics table')
)

**Status percentages (rainy days)**

Rainy Delivered % =
CALCULATE([Delivered %], 'Delivery_Logistics table'[weather] = "Rainy")

Rainy Delayed % =
CALCULATE([Delayed %], 'Delivery_Logistics table'[weather] = "Rainy")

Rainy Failed % =
CALCULATE([Failed %], 'Delivery_Logistics table'[weather] = "Rainy")

**Recommended Delivery Partner card**

Top Delivery Partner =
VAR PartnerRates =
    ADDCOLUMNS(
        VALUES('Delivery_Logistics table'[delivery_partner]),
        "Rate", [Delivered %]
    )
VAR TopRow = TOPN(1, PartnerRates, [Rate], DESC)
RETURN
    CONCATENATEX(TopRow, 'Delivery_Logistics table'[delivery_partner])
        & "  " &
    FORMAT(MAXX(TopRow, [Rate]), "0.0%")

**Best Average Rating**

VAR T = ADDCOLUMNS(VALUES('Delivery_Logistics table'[delivery_partner]),
            "RT", [Avg Rating])
VAR Best = TOPN(1, T, [RT], DESC)
RETURN CONCATENATEX(Best,
    'Delivery_Logistics table'[delivery_partner] & "  " & FORMAT([RT], "0.00"),
    UNICHAR(10), [RT], DESC)

**Troubleshooting log**

| Problem | Cause | Fix |
|--------|-----------------|-----------------|
| Stacked chart showed equal thirds for every partner | The X-axis used a measure that returned the same value for all three statuses | Replaced it with separate measures per status (Delivered %, Delayed %, Failed %) |
| UNICHAR(10) line breaks did not show in a card | The standard card ignores them | Used a Text Box with one measure per line |


**Key Business Insight**  (Created to be added as data in the text box)

•	Delhivery has the highest delivered rate overall (75.2%) and on rainy days (65.7%), and the lowest delayed rate in both conditions (19.8% and 27.7%).
•	Ecom Express has the lowest overall failure rate (4.4%).
•	Amazon Logistics has the lowest rainy-day failure rate (5.0%).
•	FedEx has the best average rating (3.70 out of 5), though ratings across partners are close (3.61 to 3.70).
•	Delivered rates fall in the rain and delayed rates rise.
•	Delhivery is the recommended delivery partner. It has the highest delivered rate in both normal and rainy conditions (75.2% and 65.7%) and the lowest delayed rate in both (19.8% and 27.7%), with a customer rating (3.69/5) close to the top.
