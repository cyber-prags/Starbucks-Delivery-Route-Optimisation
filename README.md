# Starbucks-Delivery-Route-Optimisation

## Project Overview
Imagine a time when you are operating a business and have multiple stores in an area and delivering raw materials and goods to them is a huge part of keeping the business running. Along with goods, comes the question of logistics, how many deliverymen and trucks to employ to get the job done, what routes should these trucks follow? How much fuel would taking a particular route or how much time would taking a particular route take and how much would it cost you. 

This project makes use of Graphs to solve the Travelling Salesman Problem(TSP) to optimize delivery routes for Starbucks stores in London using OpenStreetMap data and OR-Tools. The goal is to minimize the total travel time required for a single driver to deliver supplies to all Starbucks locations in the city efficiently.
This project aims 

## Features
- Extracts road network data using **osmnx**
- Computes shortest paths with **networkx**
- Uses **OR-Tools** to optimize delivery routes
- Generates interactive **Folium** and **Plotly** visualizations

---

## Installation
Ensure you have Python installed, then install the required dependencies:

```sh
pip install osmnx ortools folium plotly pandas numpy matplotlib seaborn networkx
```

---

## Dataset
The dataset `data_stores.csv` contains Starbucks store locations. The relevant columns used in the project are:
- **City**: The city where the store is located
- **Street Address**: Address of the store
- **Latitude**: Latitude coordinate
- **Longitude**: Longitude coordinate

---

## Workflow

### 1. Load and Filter Dataset
The dataset is loaded and filtered to include only Starbucks locations in London.

```python
city = "London"
dtf = pd.read_csv("data_stores.csv")
dtf = dtf[dtf["City"] == city][["City", "Street Address", "Latitude", "Longitude"]].reset_index(drop=True)
dtf = dtf.reset_index().rename(columns={"index": "id", "Latitude": "y", "Longitude": "x"})
```

### 2. Visualize Store Locations
The Starbucks locations are plotted using **Folium**, with the starting location marked in red.

```python
map = folium.Map(location=[42.99, -81.26], tiles="cartodbpositron", zoom_start=12)
data.apply(lambda row:
    folium.CircleMarker(
        location=[row["y"], row["x"]],
        color=row["color"], fill=True, radius=5).add_to(map), axis=1)
```
![image](https://github.com/user-attachments/assets/eacb5bb3-7688-4be1-a5d2-c79383196fa8)


### 3. Extract Road Network
The **osmnx** library is used to obtain a road network within a 10km radius of the starting location.

```python
G = ox.graph_from_point([42.99, -81.26], dist=10000, network_type="drive")
G = ox.add_edge_speeds(G)
G = ox.add_edge_travel_times(G)
fig, ax = ox.plot_graph(G, bgcolor="black", node_size=5, node_color="white", figsize=(16,8))
```
![image](https://github.com/user-attachments/assets/b388d9ae-d9fd-4ac7-ac5a-cdf04b90356a)

### 4. Compute Distance Matrix
A distance matrix is created using **networkx** for shortest travel times between nodes.

```python
def f(a, b):
    try:
        return nx.shortest_path_length(G, source=a, target=b, method='dijkstra', weight='travel_time')
    except:
        return np.nan

distance_matrix = np.asarray([[f(a, b) for b in dtf["node"].tolist()] for a in dtf["node"].tolist()])
distance_matrix = pd.DataFrame(distance_matrix, columns=dtf["node"].values, index=dtf["node"].values)
```

### 5. Optimize Delivery Route
**OR-Tools** is used to determine the optimal sequence of deliveries to minimize travel time.

```python
manager = pywrapcp.RoutingIndexManager(len(lst_nodes), drivers, lst_nodes.index(start_node))
model = pywrapcp.RoutingModel(manager)

def get_distance(from_index, to_index):
    return distance_matrix.iloc[from_index, to_index]

distance = model.RegisterTransitCallback(get_distance)
model.SetArcCostEvaluatorOfAllVehicles(distance)
parameters = pywrapcp.DefaultRoutingSearchParameters()
parameters.first_solution_strategy = routing_enums_pb2.FirstSolutionStrategy.PATH_CHEAPEST_ARC
solution = model.SolveWithParameters(parameters)
```

### 6. Generate Optimal Route
The optimal sequence of stores to visit is determined and visualized.

```python
index = model.Start(0)
route_idx, route_distance = [], 0
while not model.IsEnd(index):
    route_idx.append(manager.IndexToNode(index))
    previous_index = index
    index = solution.Value(model.NextVar(index))
    route_distance += get_distance(previous_index, index)
```
![image](https://github.com/user-attachments/assets/d0449a73-a4ae-4b32-91d8-0599cae14f21)


The optimal sequence of stores is printed:

```python
print("Optimal route:", route_idx)
print("Total distance of the route:", round(route_distance/1000, 2), "km")
```


---

## Results
- The shortest route for a single driver to visit all Starbucks locations in London was calculated.
- The total optimized travel distance is **6.63 km**.
- The interactive **Folium** and **Plotly** maps visualize the delivery path.

---

## Visualization
- **Folium** is used to plot nodes and the route.
- **Plotly** creates an animated map of the optimized route.

```python
fig = px.scatter_mapbox(df, lon="start_x", lat="start_y", zoom=15, width=900, height=700, animation_frame="id", mapbox_style="carto-positron")
fig.show()
```
![image](https://github.com/user-attachments/assets/069d77e6-cc5d-4aac-b866-3cfd48ecafbe)

---

## Future Improvements
- Extend the project to multiple drivers for better efficiency.
- Incorporate traffic data for real-time route adjustments.
- Automate data updates for scalability to other cities.

---

## License
This project is licensed under the **MIT License**.

---

