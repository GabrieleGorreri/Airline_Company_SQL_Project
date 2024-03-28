# Airline Company SQL Project

## INSIGHT TASKS
The purpose of these tasks is for you to work with and think about the type of data that you would use in the role and draw and explain the conclusions and recommendations you make to management. 

### TASK 1 - SQL

We have provided a simplified version of the data schema we use in the attachment. It is a star schema, with revenue transactions in the centre and descriptive dimensions linking to this.
Based on this schema please can you write some pseudocode (ideally SQL) to create summarised datasets that answer the following questions:

1. How many passengers are due to fly in the each of the next 3 months?
2. What is the average number of days between booking and flying split by booking country (based on billing address)?
3. Generate a list of mailable customer ids who have flown in the last 2 years, but don’t have any upcoming flights booked.

*Hint: All of these questions are based on flights, so you need to apply a filter based on ServiceCategory = ‘Flight’ (from the dimService table)*
