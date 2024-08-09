# Technical Test Solutions
### by Sebastian Reddy

## Q1. Market definition: Python
**You have a global database with RE transactions over the last 20 years. They are geolocated and most of them have a market name as part of their details.
Write a python pseudocode to calculate geometrical boundaries containing all transactions for each market. For simplicity, use this example with all transactions marked as “Yangzhou market”:**

**The data is in a sql table with the following structure:**  
>**Deals(DealId, DealName, DealDate, DealPrice, AssetBuildingSurface, LatCode, LngCode, CountryCode, MarketName)**

**Read from the database or import a csv and calculate the geometrical boundary; pseudocode would work just fine.**

For the geometric boundary, I am assuming that a simple rectangle (quadrangle would be more accurate given that the Earth is a sphere) containing all of the data points for a given market will suffice. The database includes the latitude and longitude values of each deal data point so the bounding quadrangle can be defined using the latitude and longitude values of each of its 4 sides (North, East, South, West). I am including an offset variable to add a small amount of extra space around the edge of the quadrangle so that the boundary does not lie directly on top of the furthest data point but rather slightly outside. The program will return, for each market, the coordinates of the 4 corners of the bounding quadrangle, starting at the North-West corner and going clockwise. Latitude is defined such that 0deg refers to the equator, positive values increase towards the North pole at +90deg (equivalent to 90N), and negative values decrease towards the South pole at -90deg (equivalent to 90S). Similarly, longitude is defined with Greenwich at 0deg, positive values increase Eastwards, and negative values decrease Westwards, both then meeting at +/-180deg (180E or 180W). I am assuming that in the database, the latitude and longitude values are stored as signed floats where the sign represents North/South and East/West.

###### Pseudocode:
```Python
# Import relevant columns from SQL database into separate python arrays
DealIds, LatCodes, LngCodes, MarketNames = query(SELECT DealId, LatCode, LngCode, MarketName FROM Deals)

# Get names of distinct markets without repetitions
Markets = distinct(MarketNames)

# Initialise output array of coordinates of corners of each market bounding box
OutputCoordinates = []


# For each market, loop over all data points and store in new arrays relevant (local) data points (i.e. those inside that market)
for market in Markets:
	LocalIds = []
	LocalLats = []
	LocalLongs = []

	for iter, val in MarketNames:
		if val == market:
			LocalIds.append(DealIds[iter])
			LocalLats.append(LatCodes[iter])
			LocalLongs.append(LngCodes[iter])

	# Find furthest lat/long value in each direction and add offset to get coordinates of each boundary
	offset = ...
	north = max(LocalLats) + offset		# Northernmost
	east = max(LocalLongs) + offset		# Easternmost
	south = min(LocalLats) - offset		# Southernmost
	west = min(LocalLongs) - offset		# Westernmost

	# Append coordinates of corners bounding box to output array. Starts in North-East corner and moves clockwise
	OutputCoordinates.append(((north, east), (north, west), (south, west), (south, east)))

# Output values
for iter in range(len(Markets)):
	print("Market: ", Markets[iter])
	print("Bounding box coordinates: ", OutputCoordinates[iter])


```


## Q2. LTV Portfolios
**You have to analyse a European last mile logistics platform. The owner is asking whether they could get more leverage on their debt. Calculate the LTV for the portfolio given, for each of 10 the properties, the following:**
- **Initial investment price -> InvPrice_1, …, InvPrice_10**
- **Current price -> CurrentPrice_1, …, CurrentPrice_10**
**Plus the total initial debt amount -> LoanAmount**

**Tables schema:**  
> **DealsTable(PropertyId, InvPrice, DealId, DealDate)**  
> **PropertyTable(PropertyId, CurrentPrice)**   
> **LoanTable(DealId, LoanId, LoanAmmount)**  

**Write a sql query to calculate the current leverage for the portfolio (LTV). How would you advise the owner?**

Loan To Value (LTV) is a defined as the ratio between the amount loaned to purchase an asset and the full value of said asset, i.e. $$LTV = \frac{Loan \ Amount}{Price}$$.  
Generally speaking, the higher the LTV ratio, the riskier a loan is seen as being, as a high LTV ratio implies that there is very little equity contained within the asset itself. Additionally, loan contracts will often stipulate a ceiling on the LTV for a particular asset/portfolio. The borrower will borrow at an LTV somewhat lower than this limit to give themselves a bit of space to allow the asset price to fluctuate without breaching the terms of the contract.  

For this particular example, there are two LTV ratios that can be calculated: the LTV at day 1 when the loan was originated, and the LTV at the current time. The LTV ratio is calculated by summing up the total loan amount borrowed for the whole portfolio and dividing it by the total price of the whole portfolio. These two ratios can be compared against each other to advise the owner.  

If the current LTV is lower than the day 1 LTV, the value of the portfolio has increased over time and the owner is in a very good position to raise more leverage. The larger the difference, the more new capital the owner can comfortably raise. However, if the current LTV is higher than the investment LTV, the value of the portfolio has decreased somewhat. In this case, it is less advisable to get more leverage but not impossible. If there has only been a small drop in value, and the current LTV is significantly below the contractual limit (if they have one), it would still be possible for the owner to raise more capital. But, the larger the change in LTV, and the closer the current LTV is to the contractual limit, the less advisable it is to try and leverage up.

###### SQL Code:
```SQL
-- Calculate initial LTV at investment
-- Sum over all loan amounts and divide by the sum over all initial investment prices
SELECT (SUM(LoanAmmount) / SUM(InvPrice)) AS InvLTV
FROM DealsTable
	JOIN LoanTable
		ON DealsTable.DealID = LoanTable.DealID;

-- Calculate current LTV in a similar manner
SELECT (SUM(LoanAmmount) / SUM(CurrentPrice)) AS CurrentLTV
FROM DealsTable
	JOIN PropertyTable
		ON DealsTable.PropertyID = PropertyTable.PropertyID
	JOIN LoanTable
		ON DealsTable.DealID = LoanTable.DealID;
```

## Q3. The TV contest
**A relevant TV station is going to launch in your country a new TV contest and you need to define some specifics about the functioning. This contest has worked well in other countries for several years and it is likely that it will stay on air for, at least, three years.  
Each day, five people will compete in two events whose results should be balanced for the final score.  
The first one consists of answering twenty questions on general knowledge. You will know the number of hits.  
The other one consists of running a 100m race on one leg. You will know the number of seconds spent for each participant.  
The best one within the five will go to a final test where they will have the last challenge to receive the prize.  
What mechanism do you consider most appropriate to select the best contestant from the tests carried out? Justify the answer.**

The difficulty in creating a fair selection criterion comes from the fact that the two contests are scored in a fundamentally different manner. The first event, the quiz, is scored using an integer that sits between 0 and 20 representing the number of correctly answered questions. This means that an individual contestant's score can be reduced down to a single percentage value that can easily be compared against the score of any other contestant. However, since there are only 20 possible scores, there is a non-negligable chance of multiple contestant achieving the same score. Which poses a problem for selecting a single individual as the winner. The most simple interpretation is to rank contestants by their score (high to low). Without additional information, it is not possible to differentiate between contestants who achieve the same score.

The race on the other hand is scored (theoretically) over the entire set of positive reals. There is no maximum time so it is not possible to calculate a percentage score for each contestant. However, there is a very low chance that two contents achieve the exact same time, it is simple to assume that this probability is effectively zero (not a bad assumption if race times are recorded to high degree of precision). This makes it easy to rank all the contestants by time (low to high). The best contestant is simply the contestant with the lowest time.

There are an enourmous number of possible scoring mechanisms but one that particularly appeals to me is to subtract a contestant's quiz score from their race time. Every contestant will compete in the quiz and recieve a score between 0 and 20. The contestants will then run the race and recieve their race time. Then, each person's quiz score (potentially multiplied by a scaling factor that can be varied to control the balance between the two contests) is subtract from their race time to give a final contest time. As in a regular race, the contestant with the smallest time wins.

The main benefit of this system is that both contests are important and a contestant can try to compensate for a bad performance in one game with a good performance in the other. But, in order to win overall a contestant has to do very well in both games. If a contestant does badly in one game, they still have a chance of doing well by excelling at the other game. This avoids a situation where a contestant fails at one game and then completely gives up on the other. It makes better TV!

**In the final phase, the host offers the best contestant the choice between a transparent box A with eight 500 euro notes and an opaque box B that could contain twice or half the money of box A with the same probability.  
Which box should the contestant choose? Justify the answer.**

The probability of box B containing more money than box A is equal to the probability that it contains less, meaning that they both have a probability of 0.5. The expected value of box A is $1.0 * 8 * 500€ = 4000€$ and the expected value of box B is $0.5 * 16 * 500€ + 0.5 * 4 * 500€ = 5000€$. So, statistically, it is better to select box B as it is 25% more valuable. Over many repetitions, if the top contestant was to always choose box B, you would expect them to earn, on average, more than if they always chose box A.

However, if the contestant is very risk adverse and feels that they must earn more than 2000€ - they can choose box A and earn 4000€ for certain, as long as they are happy missing out on the chance to earn 8000€.















