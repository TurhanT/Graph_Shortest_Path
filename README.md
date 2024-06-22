
# Graph Shortest Path Project

This project demonstrates how to use Kinetica's graph API to solve shortest path problems using SQL. The project includes a Hex notebook with visuals, showcasing the shortest routes between different points in Seattle. The exercise covers three types of routes: single source to single destination, single source to many destinations, and many sources to many destinations.

## Project Overview

### Workbook Description

This guide shows how to use Kinetica's graph API to calculate the shortest routes between different points in Seattle. The entire exercise is done using SQL. The three types of routes that we solve are:
1. Single source to single destination
2. Single source to many destinations
3. Many sources to many destinations

### Hex Notebook

The project includes a Hex notebook with visuals. You can access the notebook using the following link: [Hex Notebook](https://app.hex.tech/aebb927a-18dc-4a84-ab6a-a8dfd613029f/app/15313a68-a046-4504-b16f-5413f9e44354/latest)

## Instructions

### Connecting to Kinetica

1. **Install Kinetica Developer Edition**
   - Kinetica is a PostgreSQL database that allows you to connect it to Hex notebooks using PostgreSQL wireline.
   - Install the developer edition from [Kinetica Developer Edition](https://www.kinetica.com/developer-edition/).

2. **Connect Hex to Developer Edition**
   - Use `ngrok` to connect Hex to Kinetica.
   - Run the following command in the terminal: `ngrok tcp 5434`.
   - Update the "Host" and "Port" values in the "Kinetica Dev Edition" data browser.
   - Update the credentials under the "AUTHENTICATION" section with your Kinetica Workbench Developer Edition credentials.

### SQL Scripts

The SQL scripts included in this project cover the following steps:

1. **Insert Data Points**
    ```sql
    INSERT INTO seattle_sources 
    VALUES
    (2, 'POINT(-122.375180125237 47.8122103214264)');

    -- Display the table
    (SELECT ST_ASTEXT(wkt) AS WKT, 'Destination '||ID AS type FROM seattle_dest) UNION
    (SELECT ST_ASTEXT(wkt) AS WKT, 'Source '||ID AS type FROM seattle_sources);
    ```

2. **Solve Shortest Path for Many-to-Many Sources and Destinations**
    ```sql
    DROP TABLE IF EXISTS GRAPH_S_MANY_MANY_SOLVED;
    EXECUTE FUNCTION SOLVE_GRAPH(
        GRAPH => 'GRAPH_S',
        SOLVER_TYPE => 'SHORTEST_PATH',
        SOURCE_NODES => INPUT_TABLE(SELECT wkt AS NODE_WKTPOINT FROM seattle_sources),
        DESTINATION_NODES => INPUT_TABLE(SELECT wkt AS NODE_WKTPOINT FROM seattle_dest),
        SOLUTION_TABLE => 'GRAPH_S_MANY_MANY_SOLVED'
    );
    ```

3. **Display the Results**
    ```sql
    SELECT gms.INDEX, ST_ASTEXT(PATH) AS PATH,
        'Start From Source ' || ss.ID AS Starting_Point,
        'Destination To Destination ' || sd.ID AS Destination_Point,
        LPAD(FLOOR(gms.COST / 3600), 2, '0') || ' Hours ' ||
        LPAD(FLOOR((gms.COST % 3600) / 60), 2, '0') || ' Minutes' AS Total_Time_Cost
    FROM 
        GRAPH_S_MANY_MANY_SOLVED gms
    JOIN 
        seattle_sources ss ON ST_ALMOSTEQUALS(gms.SOURCE, ss.WKT, 2)
    JOIN 
        seattle_dest sd ON ST_ALMOSTEQUALS(gms.TARGET, sd.WKT, 2);
    ```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [Kinetica](https://www.kinetica.com/) for providing the database and graph API.
- [Hex](https://hex.tech/) for the notebook and visualization tools.
