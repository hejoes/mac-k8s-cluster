https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-snapshots.html#k8s-s3-compatible'


Executed in kibana console 
https://localhost:5601/app/dev_tools#/console

POST _snapshot/my_s3_repository
{
  "type": "s3",
  "settings": {
    "bucket": "eg-test-elastic-snapshot",
    "path_style_access": true,	
    "base_path": "eg-prod-gitlab-advanced-search", 
    "endpoint": "https://s3.teliahybridcloud.com/" 
  }
}


Create lifecycle policy - https://www.elastic.co/guide/en/elasticsearch/reference/current/slm-api-put-policy.html

PUT /_slm/policy/daily-snapshots
{
  "schedule": "0 30 1 * * ?", 
  "name": "<daily-snap-{now/d}>", 
  "repository": "my_s3_repository", 
  "config": { 
    "indices": ["data-*", "important"], 
    "ignore_unavailable": false,
    "include_global_state": false
  },
  "retention": { 
    "expire_after": "30d", 
    "min_count": 5, 
    "max_count": 50 
  }
}


working policy
GET /_slm/policy/daily-snapshots


{
  "daily-snapshot": {
    "version": 1,
    "modified_date_millis": 1668439676884,
    "policy": {
      "name": "<daily-snap-{now/d}>",
      "schedule": "0 30 6 * * ?",
      "repository": "my_s3_repository",
      "config": {
        "include_global_state": true,
        "feature_states": [],
        "ignore_unavailable": true,
        "partial": true
      },
      "retention": {
        "expire_after": "90d",
        "min_count": 3,
        "max_count": 30
      }
    },
    "next_execution_millis": 1668493800000,
    "stats": {
      "policy": "daily-snapshot",
      "snapshots_taken": 0,
      "snapshots_failed": 0,
      "snapshots_deleted": 0,
      "snapshot_deletion_failures": 0
    }
  }
}