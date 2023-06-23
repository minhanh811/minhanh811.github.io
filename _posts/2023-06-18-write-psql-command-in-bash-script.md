---
layout: post
nav: blog
title: Write psql command in Bash script
---
As I shared before in ...  Archiving a database is beneficial for almost every system but Archiving is a critical task to maintain the performance and efficiency of a database by moving older or less frequently accessed data to separate storage. By leveraging psql commands within the Bash script, we were able to automate the archiving process, ensuring the integrity and consistency of the data while minimizing manual effort.
So I tried to learn about writing psql command in Bash script.

# Step 1: I wannt to backup my database by using batches select query based on month and year. So I create a simple way to input month/year/batch size
```
#!/bin/bash

echo ""Input year to start delete:""
read year
echo ""From month:""
read from_month
if [ $from_month -lt 1 -a $from_month -gt 12  ]
then
    echo ""Invalid month""
    exit 1
fi

echo ""To month:""
read to_month
if [ $to_month -lt 1 -a $to_month -gt 12  ]
then
    echo ""Invalid month""
    exit 1
fi

if [ $from_month -gt $to_month ]
then
    echo ""Invalid month range""
    exit 1
fi

echo ""Input batch size:""
read batch_size
```
<hr>
# Step 2: Set the database connection parametters. Note: the PGPASSWORD allow you execute the psql command without asking password. I love it.
```
DB_NAME="your_database"
USER="your_username"
HOST="your_host"
PGPASSWORD="your_password"
```
<hr>
# Step 3: Copy my datab to CSV file and add all the query I executed to log file.
```
###### Backup ######

for ((month=from_month; month<=to_month; month++)); do
    if [ "$month" -eq 12 ]; then
        NEXT_MONTH=1
        NEXT_YEAR=$((year + 1))
    else
        NEXT_MONTH=$((month + 1))
        NEXT_YEAR=$year
    fi
    QUERY="SELECT id FROM orders WHERE created_at >= '$year-$month-1 00:00:00' AND created_at < '$NEXT_YEAR-$NEXT_MONTH-1 00:00:00' ORDER BY id"
    OFFSET=0
    TOTAL_COUNT=$(psql -d $DB_NAME -U $USER -h $HOST -AXqtc "SELECT COUNT(*) FROM ($QUERY) AS subquery" -t)
    CSV_COUNT=1

    # Loop until all rows are fetched
    while [ $OFFSET -lt $TOTAL_COUNT ]; do
        # Execute the psql query with batch limit and offset
        ids=`psql -d $DB_NAME -U $USER -h $HOST -AXqtc "SELECT id FROM orders WHERE created_at >= '$year-$month-1 00:00:00' AND created_at < '$NEXT_YEAR-$NEXT_MONTH-1 00:00:00' ORDER BY id OFFSET $OFFSET LIMIT $batch_size;"`

        ids_with_commas=$(echo "$ids" | paste -s -d ',' -)
        time=""${month}_${year}""

        mkdir -p backup/orders/$year/$month
        echo ""==== Backup orders ${time} ====""
        echo "SELECT id FROM orders WHERE created_at >= '$year-$month-1 00:00:00' AND created_at < '$year-$NEXT_MONTH-1 00:00:00' ORDER BY id OFFSET $OFFSET LIMIT $batch_size;" >> "backup/orders/$year/$month/orders_backup_log.txt"
        psql -d $DB_NAME -U $USER -h $HOST --echo-all -c "\copy (SELECT * FROM orders WHERE id IN ($ids_with_commas)) TO 'backup/orders/$year/$month/orders_backup_${time}_$CSV_COUNT.csv' DELIMITER ',' CSV HEADER" >> "backup/orders/$year/$month/orders_backup_log.txt"

        # Increment the offset
        OFFSET=$((OFFSET + batch_size))
        ((CSV_COUNT++))
    done

    sleep 0.5
done
```
<hr>
# Conclusion:
The script incorporated the necessary psql commands to select and export the booking records that met our archiving criteria into a CSV file. The flexibility of Bash scripting allowed us to parameterize the script, enabling customization of the archiving conditions such as date range or specific attributes. We also utilized the power of psql commands to efficiently delete the archived records from the live database, freeing up valuable storage space and optimizing query performance.
