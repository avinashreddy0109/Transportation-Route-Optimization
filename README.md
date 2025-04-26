
# 🚚 Transportation Route Optimization
# Author: Avinash Reddy Vinjamuri

# 📦 Importing Libraries
import pandas as pd
import networkx as nx
import folium
import matplotlib.pyplot as plt
import random
import numpy as np

# 📂 Load Delivery Points
# (Example: Synthetic coordinates around NYC)
delivery_points = pd.DataFrame({
    'Location': ['Warehouse', 'Client 1', 'Client 2', 'Client 3', 'Client 4', 'Client 5'],
    'Latitude': [40.7128, 40.7138, 40.7150, 40.7140, 40.7160, 40.7170],
    'Longitude': [-74.0060, -74.0050, -74.0040, -74.0070, -74.0030, -74.0020]
})

delivery_points.to_csv('data/delivery_points.csv', index=False)

# 📂 Simulated Traffic Congestion
# Higher congestion = longer travel time
traffic_congestion = pd.DataFrame({
    'Start': [],
    'End': [],
    'TrafficFactor': []
})

for i in range(len(delivery_points)):
    for j in range(len(delivery_points)):
        if i != j:
            traffic_congestion = traffic_congestion.append({
                'Start': delivery_points['Location'][i],
                'End': delivery_points['Location'][j],
                'TrafficFactor': random.uniform(1, 3)  # random traffic factor between 1 and 3
            }, ignore_index=True)

traffic_congestion.to_csv('data/traffic_congestion.csv', index=False)

# 🗺️ Create Graph
G = nx.DiGraph()

# Add nodes
for idx, row in delivery_points.iterrows():
    G.add_node(row['Location'], pos=(row['Latitude'], row['Longitude']))

# Add weighted edges based on distance * traffic factor
for idx, row in traffic_congestion.iterrows():
    start = row['Start']
    end = row['End']
    lat1 = delivery_points.loc[delivery_points['Location'] == start, 'Latitude'].values[0]
    lon1 = delivery_points.loc[delivery_points['Location'] == start, 'Longitude'].values[0]
    lat2 = delivery_points.loc[delivery_points['Location'] == end, 'Latitude'].values[0]
    lon2 = delivery_points.loc[delivery_points['Location'] == end, 'Longitude'].values[0]
    
    # Simple distance metric (Manhattan distance)
    distance = abs(lat1 - lat2) + abs(lon1 - lon2)
    
    # Weighted by traffic
    weight = distance * row['TrafficFactor']
    
    G.add_edge(start, end, weight=weight)

# 📈 Visualize Normal Route (Random Order)
normal_route = list(delivery_points['Location'])
normal_total_cost = sum(
    G[normal_route[i]][normal_route[i+1]]['weight']
    for i in range(len(normal_route) - 1)
)

# 📈 Visualize Optimized Route (Shortest Path)
optimized_route = nx.approximation.traveling_salesman_problem(G, cycle=False, weight='weight')
optimized_total_cost = sum(
    G[optimized_route[i]][optimized_route[i+1]]['weight']
    for i in range(len(optimized_route) - 1)
)

# 🌎 Create Folium Map (Normal Route)
normal_map = folium.Map(location=[40.7145, -74.0059], zoom_start=14)
for i in range(len(normal_route)-1):
    start = delivery_points.loc[delivery_points['Location'] == normal_route[i]]
    end = delivery_points.loc[delivery_points['Location'] == normal_route[i+1]]
    points = [(start['Latitude'].values[0], start['Longitude'].values[0]),
              (end['Latitude'].values[0], end['Longitude'].values[0])]
    folium.PolyLine(points, color='red', weight=2.5, opacity=1).add_to(normal_map)

normal_map.save('outputs/normal_route_map.html')

# 🌎 Create Folium Map (Optimized Route)
optimized_map = folium.Map(location=[40.7145, -74.0059], zoom_start=14)
for i in range(len(optimized_route)-1):
    start = delivery_points.loc[delivery_points['Location'] == optimized_route[i]]
    end = delivery_points.loc[delivery_points['Location'] == optimized_route[i+1]]
    points = [(start['Latitude'].values[0], start['Longitude'].values[0]),
              (end['Latitude'].values[0], end['Longitude'].values[0])]
    folium.PolyLine(points, color='green', weight=2.5, opacity=1).add_to(optimized_map)

optimized_map.save('outputs/optimized_route_map.html')

# 📊 Comparison Plot
labels = ['Normal Route', 'Optimized Route']
costs = [normal_total_cost, optimized_total_cost]

plt.figure(figsize=(8,5))
plt.bar(labels, costs, color=['red', 'green'])
plt.ylabel('Total Weighted Cost')
plt.title('Route Cost Comparison')
plt.savefig('outputs/time_comparison.png')
plt.show()

# ✅ Print Achievements
improvement = (normal_total_cost - optimized_total_cost) / normal_total_cost * 100
print(f"Total cost reduced by {improvement:.2f}% after route optimization!")
