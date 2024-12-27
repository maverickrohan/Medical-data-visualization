# Medical-data-visualization
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

# Import data
df = pd.read_csv("medical_examination.csv")

# Add 'overweight' column
df['BMI'] = df['weight'] / ((df['height'] / 100) ** 2)
df['overweight'] = (df['BMI'] > 25).astype(int)
df.drop('BMI', axis=1, inplace=True)

# Normalize data by making 0 always good and 1 always bad
df['cholesterol'] = (df['cholesterol'] > 1).astype(int)
df['gluc'] = (df['gluc'] > 1).astype(int)

# Draw Categorical Plot
def draw_cat_plot():
    # Create DataFrame for cat plot using pd.melt
    df_cat = pd.melt(df, 
                     id_vars=["cardio"], 
                     value_vars=["cholesterol", "gluc", "smoke", "alco", "active", "overweight"])
    
    # Group and reformat the data for plotting
    df_cat = df_cat.groupby(["cardio", "variable", "value"])["value"].count().reset_index(name='total')
    
    # Create the catplot
    fig = sns.catplot(x="variable", 
                      y="total", 
                      hue="value", 
                      col="cardio", 
                      data=df_cat, 
                      kind="bar").fig
    return fig

# Draw Heat Map
def draw_heat_map():
    # Clean the data
    df_heat = df[(df['ap_lo'] <= df['ap_hi']) &
                 (df['height'] >= df['height'].quantile(0.025)) &
                 (df['height'] <= df['height'].quantile(0.975)) &
                 (df['weight'] >= df['weight'].quantile(0.025)) &
                 (df['weight'] <= df['weight'].quantile(0.975))]

    # Calculate the correlation matrix
    corr = df_heat.corr()

    # Generate a mask for the upper triangle
    mask = np.triu(np.ones_like(corr, dtype=bool))

    # Set up the matplotlib figure
    fig, ax = plt.subplots(figsize=(12, 10))

    # Draw the heatmap
    sns.heatmap(corr, mask=mask, annot=True, fmt=".1f", cmap="coolwarm", ax=ax)

    return fig

# Generate the plots
cat_plot = draw_cat_plot()
cat_plot.savefig("catplot.png")

heat_map = draw_heat_map()
heat_map.savefig("heatmap.png")
