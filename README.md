# Projekt_ESS341
Question: How do the results of different methods for calculating the change of burned area, after the Wildfire in Hawaii 2023, differ form each other?

Description: For answering the questions 2 different Methods and 2 different datasets are used. From the Sentinel 2 datasets of the years 2022 (pre-incident) and 2024 (post-incident) the change in the normalized burned ratio (NBR) and the strong change, based on the cosine similarity, are calculated, plotted and compared. Then the Storng change is also calculated and plotted from the Alpha Earth (AEF) embeddings, from 2022 and 2024,  and being compared to the Strong change of the Sentinel 2 datasets.


### Data Sources:
    https://drive.google.com/drive/folders/1cnFhMAxAHG-53SUhneyHhJmdYA7DOddb

## Setup Instructions: 
    Core:
            - python=3.12
            - numpy
            - matplotlib
    
    Notebook envrionment:
            - jupyterlab
            - ipykernel
            - jupyterlab_widgets

    Geospatial stack:
            - rasterio

    other libraries:
            - os
            - matplotlib.pyplot
            - ListedColormap from matplotlib
            - TwoSlopeNorm from matplotlib
            - matplotlib.patches
            

    
    
## Execution Order: 
Every Step (e.g. 1, 2) is in a new cell. Smaller Steps (e.g. 2.1, 2.4) are run in the same cells as their first number indicates (step 2.4 is run in the same cell as step 2 etc.)
The cells are run in according to their nummerical order
x is a placeholder for the years and y is the placeholder for the band number

    Step 1: Import libraries
                    import os
                    import rasterio 
                    import numpy as np
                    import matplotlib.pyplot as plt
                    from matplotlib.colors import ListedColormap as lcm
                    from matplotlib.colors import TwoSlopeNorm as tsn
                    import matplotlib.patches as mpatches

### Sentinel 2

#### Importing and loading Sentinel2 datasets
    Step 2: Import sentinel2 datasets
            2.1 download the "S2_composite_2022" and "S2_composite_2024" datasets into a new folder called data, in your current working directory
            2.2 import os and rasterio
            2.3 write data_folder = "data"
            2.4 create a filepath called "S2_x_fp" (replace x with the respective year of the dataset), using os.path.join
                    os.path.join(data_folder, "name of dataset")
                
    Step 3: load bands of the sentinel2 datasets using rasterio
                with rasterio.open(S2_x_fp) as src:
                    img_x = src.read()
                    
    Step 4: Check data from sentinel2 datasets
            4.1 print an f-string for every year, call it "img_x"
                    print(f"img_x:")
            4.2 find out if the datasets are indeed NumPy arrays using an f-string and type()
                    print(   -Type: {type(img_x))")
            4.3 check the data type using .dtype()
                    print(f"   - Data Type: {img_x.dtype}")
            4.4 check the dimension using .ndim
                    print(f"   - Dimension: {img_x.ndim}")
            4.5 check the shape using .shape
                    print(f"   - Shape: {img_x.shape}")


#### Background map
    Step 5: make a background map

            5.1 define a normalization function:
            
                5.1.1 set vmin to 0 and vmax to 3500
                    def normalize(array, vmin=0, vmax=3500):

                5.1.2 clip to vmin and vmax
                    def normalize(array, vmin=0, vmax=3500):
                        clipped = np.clip(array, vmin, vmax)

                5.1.3 return standard normalization formula
                    def normalize(array, vmin=0, vmax=3500):
                        clipped = np.clip(array, vmin, vmax)
                        return (clipped-vmin)/(vmax-vmin)

            5.2 extract and normalize the RGB bands for both years
                for blue use the index 1 (blue is actually band 2 but because the index starts at 0 it is 1 instead)
                for green use the index 2 (green = band 3)
                for red use the index 3 (red = band 4)
                    Blue_x = normalize(img_x[1])
                    Green_x = normalize(img_x[2])
                    Red_x = normalize(img_x[3])

            5.3 stack the bands to true color composites(tcc, R=R, G=G, B=B) using np.stack
                right now our channel is at first in our array shape, but the imshow in will expect it to be at last, so we set axis = -1
                    tcc_x = np.stack((Red_x, Green_x, Blue_x), axis =-1)

            5.4 plot the backgroundmaps next to each other
                    fig, (ax1, ax2) = plt.subplots(1,2)
                    ax1.imshow(tcc_2022)
                    ax2.imshow(tcc_2024)
                give them a title and an overall title
                    fig.suptitle("Backgroundmaps")
                    ax1.set_title("TCC_2022")
                    ax2.set_title("TCC_2024")

#### NBR                
    Step 6: extract the bands 8 and 12 for each year
            comment: for the NBR calculations we only need band 8 and 12 of each year

            6.1 extract band 8 for both years using the respective img_x and their index []
                put the year at last when naming (band_8_x) to prevent invalid syntax
                using 7 for the index because the index starts with 0 instead of 1
                    band_8_x = img_x[7]

            6.2 do the same for the band 12 of each year, using 11 instead of 12 for the indey
                    band_12_x = img_x[11]

    Step 7: check if there are any NaN values for the respective bands and if yes how many
            use a f-string and np.isnan().sum
                    print(f" x band y has {np.isnan(band_y_x).xum()} NaN values")
            for x use the years and for y the bands (8 or 12)
            do this for each band we extracted in step 5

    Step 8: change NaN values of the bands to 0, using np.nan_to_num
                    band_y_x_noNaN = np.nan_to_num(band_y_x)
            for x use the years and for y the bands (8 or 12)

##### calculating NBR and Area without clipping
helps us to compare and understand different types of changes calculated with the NBR

    Step 9: calculate NBR (Normalized Burn Ratio) for both years (2022 & 2024)
            NBR = (band 8 - band 12)/(band 8 + band 12)
    
            9.1 define the calculation for the NBR
            
                9.1.1 write in the def which bands will be used
                    def calculate_NBR(band_8, band_12):
                    
                9.1.2 define the numerator of the formula
                    numerator = band_8 - band_12

                9.1.3 define the denominator of ther formula
                    denominator = band_8 + band_12

                9.1.4 create a new NumPy array for the NBR with only zeros using np.zero_like (is used as "base" for the calculations)
                    NBR = np.zeros_like(numerator, dtype= float)

                9.1.5 put together the formula with the already define parameters
                      use np.divide()
                    NBR = np.divide(numerator, denominator)

                9.1.6 assign NBR for the out in the formula
                      assign the denominator to where, so that the calculations are only perfomed when the denominator != 0
                    NBR = np.divide(numerator, denominator, out = NBR, where = denominator!=0)

                9.1.7 return NBR

            9.2 use the newly defined formula to calculate the NBR for both years
                use the bands with from the previous step
                    NBR_x = calculate_NBR(band_8_x_noNaN, band_12_x_noNaN)

    Step 10: calculate the difference of the NBR of the two years
             NBR pre - NBR post
                    diff_NBR = NBR_2022 - NBR_2024

    Step 11: calculate area for both NBRs

            11.1 define a formula for calculating the area in km2. use:
                 pixel resolution of Sentinel2 = 10m
                 pixel area in m^2 = resolution^2 
                 pixel count = num.sm()
                 convert m2 to km2

            11.2 print it as an f-string with only 2 decimals

##### Plotting NBR without clipping
    Step 12: plot one map for the NBR for each year
             plot them next to each other and give each map a Title (NBR of x) and a colorbar (fig.colorbar())
    
    Step 13: plot a map of the change in the NBR
             get rid off the lowest and highest 2%, with np.nanpercentile
             set the center (vcenter) at 0
             give the map a title and a good color
             
             comment: because we subtracted the NBR of the year 2024 from the one of 2022 (see Step 10) and the burned values are negativ, the positiv numbers are new burned areas and the negativ values are "healed" areas.

##### clipping the NBR change and calculating the new area
    Step 14: clip NBR change to only positiv values
                    diff_NBR_clipped = np.clip(diff_NBR, 0, None)

    Step 15: calculate the area of the clipped NBR change using the area formula(see Step 11)

##### Plotting the clipped NBR
    Step 16: plot the clipped NBR change

###### Categorizing and plotting NBR severity
    Step 17: categorize the clipped NBR change

            17.1 make a new array filled with NaN values, call it severity

            17.2 categorize the severities
                 low severity: 0.1<= NBR change <= 0.27
                 moderate severity: 0.27 < NBR change <= 0.66
                 high severity: 0.66 < NBR change

    Step 18: plot the categorized NBR change severities

            18.1 define cmap and choose 3 colors

            18.2 set boundaries by defining the numerical intervalls in which the classes are centered
                 classes:     1       2       3
                 bounds: 0.5  |  1.5  |  2.5  |   3.5

            18.3 define the norm by assigning the bounds and the number of colors in the cmap
                    norm = bnorm(bounds, cmap.N)

            18.4 plot the categorized map over the backgroundmap from 2022 and set class labels

#### Cosine similarity and strong change of Sentinel 2
##### Calculate cosine similarity and change
    Step 19: Handle NAN and Scale

            19.1 def a formula to scale
                    formula = ((array)/5000)-1

            19.2 get rid of the NaN values using np.nan_to_sum before the array is divided through 5000
                    def scaled(array):
                        formula =(np.nan_to_num(array)/5000)-1
                        return formula

            19.3 scale the sentinel 2 datasets of both years & call them scaled_x

    Step 20: calculate the Norm through calculating the vector length
            20.1 define the norm calculation
                 norm = np.sqrt(np.sum(array**2, axis=0))

            20.2 calculate the norms of both years
            

    Step 21: Normalize the Unit Length
             divide the scaled arrays through the normed

    Step 22: calculate the cosine similarity for the sentinel 2 datasets, by suming the multiplication of the units of both years across all bands
    cosine similarity determines the similarity of datasets based on their direction

    Step 23: calculate the cosine similarity change
             1 - cosine similarity

##### Calculate and plot strong change
    Step 24: calculate strong change

            24.1 calculate the mean and standard deviation of the cosine change, using np.nanmean and np.nanstd

            24.2 calculate z-score
                 divide cosine change minus mean by the standard deviation
            24.3 define a threshold at the 90th percentile of the z_score and create a boolean mask, name it strong_change_S2
            

    Step 25: visualize the strong change next to the change in NBR
             use the tcc_2022 as background map for the strong change and make it slightly transparent using alpha = 0.8
             make false values of the boolean mask invisible using np.where(strong_change_S2, 1, np.nan)


### AEF embeddings
#### Importing and loading AEF embeddings datasets
    Step 26: Importing AEF embeddings (similar to step 2)

            26.1 download the "AEF_embedding_2022" and "AEF_embedding_2024" datasets into your data folder, in your current working directory

            26.2 create a filepath called "AEF_x_fp" (replace x with the respective year of the dataset), using os.path.join
                    os.path.join(data_folder, "name of dataset")

    Step 27: load bands of the AEF embedding datasets using rasterio (see step 3)

    Step 28: Check data from AEF datasets
            28.1 print an f-string for every year, call it "AEF_x:"
                    print(f"AEF_x:")
            28.2 find out if the datasets are indeed NumPy arrays using an f-string and type()
                    print(   -Type: {type(AEF_x))")
            28.3 check the data type using .dtype()
                    print(f"   - Data Type: {AEF_x.dtype}")
            28.4 check the dimension using .ndim
                    print(f"   - Dimension: {AEF_x.ndim}")
            28.5 check the shape using .shape
                    print(f"   - Shape: {AEF_x.shape}")
            28.6 check for NaN values np.isnan(AEF_x).sum()
                    print(f"   - NaN values:{np.isnan(AEF_x).sum()}")

#### Strong change for AEF embeddings datasets
##### Calculate cosine similarity and change
    Step 29: Handle NaN and Scale using the formula we dfined in Step 14

    Step 30: calculate the Norm through calculating the vector length using the defined formula from Step 15

    Step 31: Normalize the Unit Length
             divide the scaled arrays through the normed

    Step 32: calculate the cosine similarity for the AEF embedding datasets, by suming the multiplication of the units of both years across all bands

    Step 33: calculate the cosine similarity change
             1 - cosine similarity

##### Calculate and plot strong change
    Step 34: calculate strong change

            34.1 calculate the mean and standard deviation of the cosine change, using np.nanmean and np.nanstd

            34.2 calculate z-score
                 divide cosine change minus mean by the standard deviation
            34.3 define a threshold at the 90th percentile of the z_score and create a boolean mask, name it strong_change_AEF

    Step 35: visualize the strong change
             use the tcc_2022 as background map for the strong change and make it slightly transparent using alpha = 0.8
             make false values of the boolean mask invisible using np.where(strong_change_AEF, 1, np.nan)


### Comparison
    Step 36: Comparison
             plot the strong change of the Sentinel2 are AEF next to each other

    Step 37: lay the two maps of the strong change over each other
             lay the Sentinel2 over the AEF one
             give them two different colors so it's easier to compare
             make a legend for both colors using mpatches.Patch
