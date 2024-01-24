-- how many merge and open and approved in repo
SELECT
    git_project_id,
    SUM(CASE WHEN status = 'merge' THEN 1 ELSE 0 END) AS merged_count,
    SUM(CASE WHEN status = 'open' THEN 1 ELSE 0 END) AS opened_count,
    SUM(CASE WHEN status = 'approved' THEN 1 ELSE 0 END) AS approved_count
FROM vc_reports
WHERE (event_name = 'merge_request')
  AND (operation_id, time_stamp) IN (
    SELECT
        operation_id,
        MAX(time_stamp)
    FROM vc_reports
    WHERE event_name = 'merge_request'
      AND (git_project_id IN ('51682728', '44269342') )
      AND (time_stamp >= '2023-10-29' AND time_stamp <= '2023-12-29')
    GROUP BY operation_id )
group by git_project_id;


-- how many merge and open and approved in branch
SELECT
    SUM(CASE WHEN status = 'merge' THEN 1 ELSE 0 END) AS merged_count,
    SUM(CASE WHEN status = 'open' THEN 1 ELSE 0 END) AS opened_count,
    SUM(CASE WHEN status = 'approved' THEN 1 ELSE 0 END) AS approved_count
FROM vc_reports
WHERE (event_name = 'merge_request')
  AND (operation_id, time_stamp) IN (
    SELECT
        operation_id,
        MAX(time_stamp)
    FROM vc_reports
    WHERE event_name = 'merge_request'
      AND (target_branch IN ('testing'))
      AND (git_project_id = '51682728' )
      AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
    GROUP BY operation_id );




-- get all still open marge-request and assaigned to whom and who publiched it
SELECT
    r.status,
    r.target_branch,
    r.source_branch,
    r.git_developer_id AS publisher_id,
    r.git_assignee_id As reviewer_id,
    r.developer_id AS developer_id,
      r.assignee_id As assigned_id,
    r.time_stamp AS timestamp
FROM vc_reports r
WHERE r.event_name = 'merge_request'
  AND r.status = 'open'
  AND r.operation_id IN (
    SELECT MAX(operation_id) AS max_operation_id
    FROM vc_reports
    WHERE event_name = 'merge_request'
      AND (git_project_id = '51682728' )
      AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
    GROUP BY git_project_id
);

-- get all  developer assaigned marge-request  and who publiched it
SELECT
    r.status,
    r.target_branch,
    r.git_developer_id AS publisher_id,
    r.git_assignee_id As reviewer_id,
    u.user_name AS reviewer_name,
    r.time_stamp AS timestamp
FROM vc_reports r
         JOIN users u ON r.assignee_id = u.id
WHERE r.event_name = 'merge_request'
  AND r.status = 'open'
  AND r.operation_id IN (
    SELECT MAX(operation_id) AS max_operation_id
    FROM vc_reports
    WHERE event_name = 'merge_request'
      AND (git_project_id = '51682728' )
      AND (git_assignee_id = '18458401')
      AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
    GROUP BY git_project_id
);

-- get all still open marge-request and assaigned to whom and who publiched it to team
SELECT
    r.status,
    r.git_project_id,
    r.target_branch,
    r.source_branch,
    r.git_developer_id AS publisher_id,
    r.git_assignee_id As reviewer_id,
    r.developer_id AS developer_id,
    r.assignee_id As assigned_id,
    r.time_stamp AS timestamp
FROM vc_reports r
WHERE r.event_name = 'merge_request'
  AND r.status = 'open'
  AND r.operation_id IN (
    SELECT MAX(operation_id) AS max_operation_id
    FROM vc_reports
    WHERE event_name = 'merge_request'
      AND (team_id = 1 )
      AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
    GROUP BY operation_id
);

-- how many merge and open and all team
SELECT
    SUM(CASE WHEN status = 'merge' THEN 1 ELSE 0 END) AS merged_count,
    SUM(CASE WHEN status = 'open' THEN 1 ELSE 0 END) AS opened_count,
    SUM(CASE WHEN status = 'approved' THEN 1 ELSE 0 END) AS approved_count
FROM vc_reports
WHERE (event_name = 'merge_request')
  AND (operation_id, time_stamp) IN (
    SELECT
        operation_id,
        MAX(time_stamp)
    FROM vc_reports
    WHERE event_name = 'merge_request'
      AND (team_id = 1 )
      AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
    GROUP BY operation_id );


SELECT r.developer_id , COUNT(*) AS merged_count
FROM vc_reports r
WHERE r.event_name = 'merge_request'
  AND r.status = 'merge'
  AND (r. git_project_id = '51682728' )
  AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
GROUP BY developer_id
ORDER BY merged_count DESC
LIMIT 10 ;

WITH LatestStatus AS (
    SELECT
        MAX(time_stamp) AS latest_time_stamp,
        operation_id
    FROM vc_reports
    WHERE status = 'merge'
    GROUP BY operation_id
)
SELECT platform_id, COUNT(*) AS merge_request_count
FROM
    vc_reports r
        JOIN
    LatestStatus ls ON r.operation_id = ls.operation_id AND r.time_stamp = ls.latest_time_stamp
WHERE
        r.status = 'merge'
GROUP BY
    platform_id
ORDER BY
    merge_request_count DESC
LIMIT
    5;


SELECT
    DISTINCT ON (operation_id)
    platform_id,
    COUNT(*) AS merge_request_count
FROM
    vc_reports
WHERE
        status = 'merge'
group by platform_id
ORDER BY
    operation_id, time_stamp DESC
LIMIT
    5;





SELECT artifact_id , COUNT(*) AS issuecount
FROM jira_webhook_issue join artifacts on artifact_id = artifacts.id
WHERE status != 'Done'
GROUP BY artifact_id
ORDER BY artifact_id;


SELECT id, artifact_id, assignee_account_id, issue_type, jira_issue_id, jira_issue_key, jira_project_id, jira_project_key, last_update_time_stamp, priority, start_tracking_time_stamp, status, summary, team_id, assignee_user_id, last_update_user_id
FROM public.jira_webhook_issue
WHERE team_id = 1;


SELECT id, artifact_id, assignee_account_id, issue_type, jira_issue_id, jira_issue_key, jira_project_id, jira_project_key, last_update_time_stamp, priority, start_tracking_time_stamp, status, summary, team_id, assignee_user_id, last_update_user_id
FROM public.jira_webhook_issue
WHERE assignee_user_id = 2;


SELECT artifact_id, COUNT(*) AS issueCount
FROM jira_webhook_issue
WHERE status != 'Done' AND issue_type = 'Bug'
AND last_update_time_stamp IS NOT NULL AND last_update_time_stamp BETWEEN '2023-10-30' AND '2023-11-30'
GROUP BY artifact_id
ORDER BY artifact_id


SELECT
    DATE(last_update_time_stamp) as date,
    COUNT(*) as num_of_issues
FROM
    jira_webhook_issue
WHERE
        status = 'Done'
GROUP BY
    DATE(last_update_time_stamp)
ORDER BY
    DATE(last_update_time_stamp);




SELECT
    r.status,
    r.git_project_id,
    r.target_branch,
    r.source_branch,
    r.git_developer_id AS publisher_id,
    r.git_assignee_id As reviewer_id,
    r.developer_id AS developer_id,
    r.assignee_id As assigned_id,
    r.time_stamp AS timestamp
FROM vc_reports r
WHERE r.event_name = 'merge_request'
  AND ( r.status = 'open' OR r.status = 'approved')
  AND (operation_id, time_stamp) IN (
    SELECT
        operation_id,
        MAX(time_stamp)
    FROM vc_reports
    WHERE event_name = 'merge_request'
      AND (team_id = 1 )
      AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
    GROUP BY operation_id
);


SELECT team_id, COUNT(*) AS merge_count
FROM ( SELECT operation_id, team_id, MAX(time_stamp) AS last_timestamp
         FROM vc_reports
         WHERE event_name = 'merge_request'
           AND status = 'merge'
           AND team_id IN (2)
           AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
         GROUP BY operation_id, team_id
     ) AS LastStatusPerOperation
GROUP BY team_id
ORDER BY merge_count DESC
LIMIT 5;


SELECT
    developer_id, COUNT(*) AS merge_count
FROM ( SELECT operation_id, developer_id, MAX(time_stamp) AS last_timestamp
         FROM vc_reports
         WHERE event_name = 'merge_request'
           AND status = 'merge'
           AND developer_id IN (1)
           AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
         GROUP BY operation_id, developer_id
     ) AS LastStatusPerOperation
GROUP BY developer_id
ORDER BY merge_count DESC
LIMIT 5;

SELECT
    git_project_id,
    platform_id,
    COUNT(*) AS merge_count
FROM (
         SELECT
             operation_id,
             git_project_id,
             platform_id,
             MAX(time_stamp) AS last_timestamp
         FROM
             vc_reports
         WHERE
                 event_name = 'merge_request'
           AND status = 'merge'
           AND platform_id IN (5)
           AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
         GROUP BY
             operation_id, git_project_id, platform_id
     ) AS LastStatusPerOperation
GROUP BY
    git_project_id, platform_id
ORDER BY
    merge_count DESC
LIMIT 5;


SELECT
    SUM(CASE WHEN status = 'merge' THEN 1 ELSE 0 END) AS merged_count,
    SUM(CASE WHEN status = 'open' THEN 1 ELSE 0 END) AS opened_count,
    SUM(CASE WHEN status = 'approved' THEN 1 ELSE 0 END) AS approved_count
FROM vc_reports
WHERE (event_name = 'merge_request')
  AND (operation_id, time_stamp) IN (
    SELECT
        operation_id,
        MAX(time_stamp)
    FROM vc_reports
    WHERE event_name = 'merge_request'
      AND (git_project_id = '51682728' )
      AND (target_branch = 'main' )
      AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
    GROUP BY operation_id  );


SELECT
    r.target_branch AS branch_name,
    SUM(CASE WHEN r.status = 'merge' THEN 1 ELSE 0 END) AS merged_count,
    SUM(CASE WHEN r.status = 'open' THEN 1 ELSE 0 END) AS opened_count,
    SUM(CASE WHEN r.status = 'approved' THEN 1 ELSE 0 END) AS approved_count
FROM vc_reports r
WHERE r.event_name = 'merge_request'
  AND (r.operation_id, r.time_stamp) IN (
    SELECT
        operation_id,
        MAX(time_stamp)
    FROM vc_reports
    WHERE event_name = 'merge_request'
      AND r.target_branch IN ('main','test')
      AND (git_project_id = '51682728' )
      AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
    GROUP BY operation_id
)
GROUP BY r.target_branch;


SELECT
    SUM(CASE WHEN status = 'merge' THEN 1 ELSE 0 END) AS merged_count,
    SUM(CASE WHEN status = 'open' THEN 1 ELSE 0 END) AS opened_count,
    SUM(CASE WHEN status = 'approved' THEN 1 ELSE 0 END) AS approved_count,
    git_project_id
FROM vc_reports
WHERE (event_name = 'merge_request')
  AND (operation_id, time_stamp) IN (
    SELECT
        operation_id,
        MAX(time_stamp)
    FROM vc_reports
    WHERE event_name = 'merge_request'
      AND (git_project_id = '51682728' )
      AND (time_stamp BETWEEN '2023-10-29' AND '2023-12-29')
    GROUP BY operation_id
)
GROUP BY  git_project_id;
