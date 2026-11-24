# Opening the Power BI report

`MelbourneRoadRepairs.pbip` is a Power BI Project: the report (`MelbourneRoadRepairs.Report`, PBIR format) and the semantic model (`MelbourneRoadRepairs.SemanticModel`, TMDL) saved as text, so the repo can show every change. Power BI Desktop opens it like a `.pbix`.

## What you need

- Power BI Desktop for Windows, a recent version (2026). If Desktop is older than about mid 2025, turn on **File, Options and settings, Options, Preview features, Power BI Project (.pbip) save option** and **Store reports using enhanced metadata format (PBIR)**, then restart Desktop.
- The repo folder on the Windows machine. In Parallels the Mac home folder is usually shared as `\\Mac\Home`.

## Open it

1. In Windows, go to the repo's `powerbi` folder and double-click `MelbourneRoadRepairs.pbip`.
2. The visuals are empty the first time. Select **Refresh** on the Home ribbon.
   - By default the data is read from the repo's public copy on GitHub (`DataFolder` is `https://raw.githubusercontent.com/cjmuldoon/plotlay-melbourne-road-repairs/main/data/`). If Desktop asks how to connect, choose **Anonymous** and then **Connect**.
   - To read the files from disk instead, go to **Home, Transform data, Edit parameters** and set `DataFolder` to the full path of the repo's `data` folder, **ending with a backslash**. Then select **OK** and **Apply changes**.
   - If Desktop warns about privacy levels, choose **Ignore privacy levels** (all the sources are public open data).
3. Check the numbers (below).

If the report was updated while open, close Power BI and reopen the .pbip rather than using Apply external changes.

## Check the numbers after Refresh

After **Refresh**:

| Page | Card | Should read |
|---|---|---|
| Priority | Segments scored | 7,483 |
| Priority | Median score, 0 to 100 | 50.7 |
| Priority | Share High or Very high | 18% |
| Drivers and repairs | Score of the segment picked | 85.3 (RD-5955, Intersection of Bowen Crescent and Garton Street, as it opens) |
| Drivers and repairs | Its rank of 7,483 | 1 |
| Drivers and repairs | Road repairs raised (the top 50) | 50 |
| Drivers and repairs | Their cost at the rates (indicative) | $1,521,568 |

With **Road type** on all (as it opens), the class chart reads Very low 167, Low 1,249, Medium 4,756, High 1,301, Very high 10; the Shape Map draws the 1,311 High and Very high segments. The table of the 10 highest starts with Intersection of Bowen Crescent and Garton Street (85.3) and ends with Intersection of Epsom Road and Kensington Road (80.1).

On **Drivers and repairs**, the segment picked shows condition 93, road type 70, near a school or hospital 76 and surface age 89. The treatment table: 22 reconstructions over 11,031 m² ($1,058,976) and 28 resurfacings over 17,792 m² ($462,592). The sample table lists 24 made-up assets.

## The pages

- **Priority**: the High and Very high segments on a Shape Map, the number of segments scored, the median score, the share High or Very high, the segments in each class and the 10 highest scores. The **Road type** slicer opens on every road type.
- **Drivers and repairs**: the four drivers of the segment picked in the **Segment** slicer (it opens on the first, RD-5955), the drivers averaged by class for all segments, the 50 road repairs by treatment with their cost at the rates, and the building, roof and gutter inspection, which is made-up sample data. The slicer changes only the picked segment's cards and chart.
- **Every segment**: all 7,483 segments on Azure Maps, coloured by class.

## The model

- **Segments**: one row per rated road segment (`data/road_segments.csv`), keyed by `key` (RD-0001...), with its condition, road type, surface, area, distance to a school or hospital, surface age, `priority_score`, `priority_class`, `priority_rank` and the four drivers. Power Query adds `class_order` (the classes sort Very low to Very high) and `segment` (key and description, for the slicer). Measures: `Segments scored`, `Median score`, `Share high or very high`, `Top score`, `Rank shown`, `Class code`.
- **Drivers**: the four `driver_*` columns of the same file unpivoted in Power Query, one row per segment and driver, each 0 to 100 (100 raises the priority most). Measure `Driver value`.
- **Centres**: a point inside each segment (`data/geometry/road_segment_centres.csv`), for Azure Maps.
- **Repairs** (`data/repairs.csv`, with a `treatment` name made in Power Query from `code`): the 50 road repairs and the 12 sample ones. Measures `Repair count`, `Repair area m2`, `Repair cost` (quantity times the price book rate, before the estimate's allowances and GST).
- **Sample** (`data/sample_inspection.csv`): the made-up sample of 24 assets at 8 sample sites.

## The two maps

- **Shape Map** (Priority). The custom map is saved inside the report (`MelbourneRoadRepairs.Report/StaticResources/RegisteredResources/`), cut from `data/geometry/road_segments.topojson` to the 1,311 High and Very high segments: a Shape Map draws about 1,500 locations and samples the rest, so it cannot take all 7,483. The visual has a filter on `priority_class` to match. The file also carries the context, since the Shape Map has no base map: the suburbs round the city (ABS, CC BY 4.0) in a pale ground tone, the Yarra and the harbour as the pale blue background (Vicmap Hydro, CC BY 4.0), the City of Melbourne boundary as a line and every rated segment as a faint outline. If the map shows the United States instead (an older Desktop can ignore a saved map): select it, then **Format visual**, **Visual**, **Map settings**, **Map type: Custom map**, **Add map type**, and choose `..\data\geometry\road_segments.topojson` (that file has every segment and no context). If the Shape Map is not in the Visualizations pane, turn it on in **File, Options and settings, Options, Preview features, Shape map visual** and restart Desktop.
- **Azure Maps** (Every segment). It opens on a light grey street map zoomed to the points. Location is `Centres[key]`, latitude and longitude are averaged (one point per segment), the legend is the priority class. Your organisation's admin must allow Azure Maps (on by default on a personal account). Power BI's PDF export does not wait for the Azure base map, so take a screenshot of this page instead.

If a map visual comes up empty, drag the fields back:
- Shape Map: Location `Segments[key]`, Color saturation `[Class code]`.
- Azure Maps: Location `Centres[key]`, Latitude `Centres[lat]` (Average), Longitude `Centres[lng]` (Average), Legend `Segments[priority_class]`.

Then save with **File, Save**, and if you like **File, Save as** a `.pbix`, which includes the data.

## Publish (only if you choose to)

Use **Home, Publish**. In the Power BI service, set the semantic model's **Data source credentials** for the GitHub address to **Anonymous**. Scheduled refresh then works with no gateway. **Publish to web** makes a public link, so only use it if you mean the report to be public.

## What the numbers are

The score is a prioritisation aid with stated weights, not an engineering assessment: condition 50% (on Plotlay's scale, 1 very good to 5 very poor, higher raises the priority; it is 6 minus the council's rating, 1 failed to 5 good, which `condition_council` keeps), road type 20%, nearness to a school or hospital 15% (0 to 600 m, closer raises the priority), surface age 15%, each rescaled 0 to 100 and averaged with those weights. Move the weights and the order changes; the notebook in `python/` shows by how much. Costs are indicative: typical published council rates, not quotes. The building, roof and gutter inspection is a made-up sample that shows the workflow, not real buildings.

## Screenshots

Capture at 1920 × 1080 (Windows display scaling 100%), with the Plotlay theme, and save them to `powerbi/Screenshots/`. Keep the footer with the attribution visible in every shot.
