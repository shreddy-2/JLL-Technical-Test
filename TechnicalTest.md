# Technical Test Solutions
### by Sebastian Reddy

## Q1. Market definition: Python
**You have a global database with RE transactions over the last 20 years. They are geolocated and most of them have a market name as part of their details.
Write a python pseudocode to calculate geometrical boundaries containing all transactions for each market. For simplicity, use this example with all transactions marked as “Yangzhou market”:**

**The data is in a sql table with the following structure:**  
>**Deals(DealId, DealName, DealDate, DealPrice, AssetBuildingSurface, LatCode, LngCode, CountryCode, MarketName)**

**Read from the database or import a csv and calculate the geometrical boundary; pseudocode would work just fine.**

For the geometric boundary, I am assuming that a simple rectangle (quadrangle would be more accurate given that the Earth is a sphere) containing all of the data points for a given market will suffice. The database includes the latitude and longitude values of each deal data point so the bounding quadrangle can be defined using the latitude and longitude values of each of its 4 sides (North, East, South, West). I am including an offset variable to add a small amount of extra space around the edge of the quadrangle so that the boundary does not lie directly on top of the furthest data point but rather slightly outside. The program will return, for each market, the coordinates of the 4 corners of the bounding quadrangle, starting at the North-West corner and going clockwise. Latitude is defined such that 0deg refers to the equator, positive values increase towards the North pole at +90deg (equivalent to 90N), and negative values decrease towards the South pole at -90deg (equivalent to 90S). Similarly, longitude is defined with Greenwich at 0deg, positive values increase Eastwards, and negative values decrease Westwards, both then meeting at +/-180deg. I am assuming that in the database, the latitude and longitude values are stored as signed floats where the sign represents North/South and East/West.

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

	# Append coordinates of bounding box to output array. Starts in North-East corner and moves clockwise
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
**Plus the total initial debt amount -> LoanAmmount**

**Tables schema:**  
- **DealsTable(PropertyId, InvPrice, DealId, DealDate)**
- **PropertyTable(PropertyId, CurrentPrice)** 
- **LoanTable(DealId, LoanId, LoanAmmount)**

**Write a sql query to calculate the current leverage for the portfolio (LTV). How would you advise the owner?**

Loan To Value (LTV) is a defined as the ratio between the amount loaned to purchase an asset and the full value of said asset. In this case, $LTV = \frac{Initial Investment Price}{Current Price}$. 























