To check if a column exists in table ccp_values;

`SHOW COLUMNS FROM ccp_values`

After confirming a column exists, to see **all values** from the column eg. st_steam_mass_flow_rate from the table:

`SELECT st_steam_mass_flow_rate FROM ccp_values;` 


To see recent values eg. last 10 rows only:
    
`SELECT timestamp, st_steam_mass_flow_rate FROM ccp_values ORDER BY timestamp, DESC LIMIT 10;`