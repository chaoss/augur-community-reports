# Queries:

Sequence of discourse types
```sql
SELECT A.issue_id,
	b.msg_id,
	b.msg_timestamp,
	b.repo_id, 
	C.* 
FROM
	issue_message_ref A,
	message b,
	( SELECT msg_id, discourse_act, MAX ( data_collection_date ) 
		FROM discourse_insights GROUP BY discourse_insights.msg_id, discourse_insights.discourse_act ) C 
WHERE
	A.msg_id = b.msg_id 
	AND C.msg_id = A.msg_id 
	AND b.msg_id = C.msg_id
	ORDER BY b.repo_id, A.issue_id, b.msg_timestamp
	;
	
-- for each issue_id, msg_id, repo_id triple, you have the sequence of discourse acts by 
-- issue for a repo.
```

Discourse breadth:
```sql

select repo_id, count(*) as records,
round(sum(answer_pct)/count(*),4) as answers,
round(sum(question)/count(*),4) as question, 
round(sum(announcement)/count(*),4) as announcement, 
round(sum(appreciation)/count(*),4) as appreciation, 
round(sum(elaboration)/count(*),4) as elaboration, 
round(sum(humor)/count(*),4) as humor, 
round(sum(agreement)/count(*),4) as agreement, 
round(sum(disagreement)/count(*),4) as disagreement, 
round(sum(negative_reaction)/count(*),4) as negative_reaction,
message_total
  from 
(
select message.msg_id, message.repo_id, 
round((answer::numeric/message_total::numeric)::numeric,3) as answer_pct,
round((question::numeric/message_total::numeric)::numeric,3) as question,
round((announcement::numeric/message_total::numeric)::numeric,3) as announcement,
round((appreciation::numeric/message_total::numeric)::numeric,3) as appreciation,
round((elaboration::numeric/message_total::numeric)::numeric,3) as elaboration,
round((humor::numeric/message_total::numeric)::numeric,3) as humor,
round((agreement::numeric/message_total::numeric)::numeric,3) as agreement,
round((disagreement::numeric/message_total::numeric)::numeric,3) as disagreement,  
round((negative_reaction::numeric/message_total::numeric)::numeric,3) as negative_reaction,
message_total 
from (
	SELECT
			discourse_insights.msg_id,
			COUNT ( * ) FILTER ( WHERE discourse_act = 'answer' ) AS answer,
			COUNT ( * ) FILTER ( WHERE discourse_act = 'question' ) AS question,
			COUNT ( * ) FILTER ( WHERE discourse_act = 'announcement' ) AS announcement,
			COUNT ( * ) FILTER ( WHERE discourse_act = 'appreciation' ) AS appreciation,
			COUNT ( * ) FILTER ( WHERE discourse_act = 'elaboration' ) AS elaboration,
			COUNT ( * ) FILTER ( WHERE discourse_act = 'humor' ) AS humor,
			COUNT ( * ) FILTER ( WHERE discourse_act = 'agreement' ) AS agreement,
			COUNT ( * ) FILTER ( WHERE discourse_act = 'disagreement' ) AS disagreement,
			COUNT ( * ) FILTER ( WHERE discourse_act = 'negativereaction' ) AS negative_reaction,
			c.message_total as message_total 
		FROM
			discourse_insights, 
				(select msg_id, count(*) as message_total from discourse_insights group by msg_id order by message_total) c
		WHERE discourse_insights.msg_id=c.msg_id 
		GROUP BY
			discourse_insights.msg_id,
			c.message_total
		ORDER BY
			msg_id) D, message 
			where d.msg_id=message.msg_id
			order by repo_id, msg_id ) X group by X.repo_id, X.message_total
```