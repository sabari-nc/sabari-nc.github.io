---
title: "How I Trick Python Into Making Fancy Maps"
date: 2025-09-29
layout: post
author_profile: true
image: /images/blog/study-area-map.jpg
---

Every research project needs that one “study area map” everyone keeps asking for. You know, the one with hills in the background, boundaries neatly drawn, points sprinkled on top, and a compass just to look professional.

Usually, making that map means opening up GIS software, dragging layers around, changing colors, resizing labels. I decided to make Python do it instead.

## What you need

- **DEM file** (`dem.tif`) — your hills and valleys.
- **District boundary** (`district.shp`) — your area outline.
- **Optional**: landslide points (`landslides.gpkg`), station points (`stations.shp`).
- **Optional**: a folder called `layers/` for extras like roads or rivers.

## The recipe

Copy the code below into a `.py` file or a notebook cell.

```python
import rasterio, numpy as np, geopandas as gpd, matplotlib.pyplot as plt
from matplotlib.lines import Line2D
from matplotlib.colors import Normalize
from pathlib import Path
from cmcrameri import cm as cmc

# ---- Make hills look 3D with fake sunlight ----
def hillshade(arr, azimuth=315, altitude=45, dx=None, dy=None):
    xg, yg = np.gradient(arr, dx, dy)
    slope = np.pi/2. - np.arctan(np.hypot(xg, yg))
    aspect = np.arctan2(-xg, yg)
    az, alt = np.deg2rad(azimuth), np.deg2rad(altitude)
    return np.sin(alt)*np.sin(slope) + np.cos(alt)*np.cos(slope)*np.cos(az - aspect)

# ---- Draw a scalebar so people know distances ----
def add_scalebar(ax, xmin, xmax, ymin, ymax):
    def nice_step(r):
        raw=r/5.0; exp=np.floor(np.log10(raw)); base=raw/(10**exp)
        return (1 if base<=1.5 else 2 if base<=3.5 else 5 if base<=7.5 else 10)*(10**exp)
    map_w, map_h = xmax-xmin, ymax-ymin
    half_len = nice_step(map_w); total_len=2*half_len
    x0, y0 = xmin+0.03*map_w, ymin+0.05*map_h; bar_h=0.01*map_h
    ax.add_patch(plt.Rectangle((x0,y0),half_len,bar_h,facecolor='white',edgecolor='black'))
    ax.add_patch(plt.Rectangle((x0+half_len,y0),half_len,bar_h,facecolor='black',edgecolor='black'))
    for i,m in enumerate([0,total_len/2,total_len]):
        ax.text(x0+i*half_len,y0-0.015*map_h,str(int(m/1000)),ha='center',va='top',fontsize=9,color='white')
    ax.text(x0+total_len/2,y0+0.022*map_h,"km",ha='center',va='bottom',fontsize=9,color='white')

# ---- The chef: cooks the map using your files ----
def make_study_area_map(dem_path, district_path,
                        events_path=None, stations_path=None,
                        extra_vectors_dir=None,
                        out_folder="outputs", epsg=32643,
                        pad_m=5000, colormap="vik"):

    out_folder=Path(out_folder); out_folder.mkdir(parents=True,exist_ok=True)

    # Load district outline
    district=gpd.read_file(district_path).to_crs(epsg=epsg)

    # Add some breathing space around the district
    xmin,ymin,xmax,ymax=district.total_bounds
    xmin-=pad_m; xmax+=pad_m; ymin-=pad_m; ymax+=pad_m

    # Load the DEM (the hill photo)
    with rasterio.open(dem_path) as src:
        dem=src.read(1).astype("float32")
        if src.nodata is not None: dem[dem==src.nodata]=np.nan
        dem_bounds, res = src.bounds, src.res

    # Shade the hills to look fancy
    hs=hillshade(dem,dx=res[0],dy=res[1])
    hs_norm=(hs-np.nanmin(hs))/(np.nanmax(hs)-np.nanmin(hs)+1e-12)
    cmap=getattr(cmc,colormap,plt.cm.get_cmap(colormap))
    norm=Normalize(vmin=float(np.nanmin(dem)),vmax=float(np.nanmax(dem)))
    elev_rgba=cmap(norm(dem)); elev_rgba[np.isnan(dem),:]=[0,0,0,0]
    shaded=np.clip(elev_rgba[...,:3]*(0.4+0.6*hs_norm[...,None]),0,1)

    # Start cooking the map
    fig,ax=plt.subplots(figsize=(12,8))
    ax.imshow(shaded,extent=(dem_bounds.left,dem_bounds.right,dem_bounds.bottom,dem_bounds.top),origin="upper")
    district.plot(ax=ax,facecolor="none",edgecolor="black",lw=1.5)

    # Add events if you have them
    if events_path:
        gpd.read_file(events_path).to_crs(epsg=epsg).plot(ax=ax,marker="o",color="red",markersize=30,edgecolor="k")
    # Add stations if you have them
    if stations_path:
        gpd.read_file(stations_path).to_crs(epsg=epsg).plot(ax=ax,marker="^",color="green",edgecolor="k",markersize=80)
    # Throw in anything from the extra folder
    if extra_vectors_dir:
        for f in Path(extra_vectors_dir).glob("*.*"):
            try: gpd.read_file(f).to_crs(epsg=epsg).plot(ax=ax,alpha=0.7)
            except: pass

    ax.set_xlim(xmin,xmax); ax.set_ylim(ymin,ymax)
    ax.set_xlabel("Longitude"); ax.set_ylabel("Latitude")

    # Legend so people know what’s what
    handles=[Line2D([0],[0],color="black",lw=2,label="District"),
             Line2D([0],[0],marker="o",color="k",markerfacecolor="red",markersize=8,label="Events"),
             Line2D([0],[0],marker="^",color="k",markerfacecolor="green",markersize=10,label="Stations")]
    ax.legend(handles=handles,loc="lower left",frameon=True)

    # Add scalebar + north arrow
    add_scalebar(ax,xmin,xmax,ymin,ymax)
    ax.annotate('',xy=(0.05,0.9),xytext=(0.05,0.8),xycoords='axes fraction',
                arrowprops=dict(facecolor='black',width=3,headwidth=10))
    ax.text(0.05,0.92,'N',transform=ax.transAxes,ha='center',fontsize=12,fontweight='bold')

    ax.set_title("Study Area Map",fontsize=14,fontweight="bold")

    # Save the dish
    fig.savefig(out_folder/"study_area_map.png",dpi=300,bbox_inches="tight")
    fig.savefig(out_folder/"study_area_map.pdf",dpi=300,bbox_inches="tight")
    plt.close(fig)