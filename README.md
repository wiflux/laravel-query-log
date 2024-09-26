# laravel-query-log
This document shows you how to enable query logs and write them into new file.

#### Step1 :

add env variable in `.env`
```
QUERY_LOG=true
```
#### Step2 :
add new veriable in `config/app.php`.
```
'query_log' => (bool) env('QUERY_LOG', false),
```

#### Step3 :
add new logging channel in `config/logging.php`.

```
'channels' => [
        ....
        ....
        ....
        'query' => [
                    'driver' => 'single',
                    'path' => storage_path('logs/queries.log'),
                    'level' => 'info',
                    'replace_placeholders' => true,
                ],
        ]
```

#### Step4 :
add code in `app/Providers/AppServiceProvider.php`.

```
  public function boot(): void
    {
        if (config('app.query_log') == true) {
            DB::listen(function($query) {
                Log::channel('query')->info(
                    Str::replaceArray('?', $query->bindings, $query->sql),
                    [
                    'time' => $query->time //in milliseconds
                    ]
                );
            });
        }

        ...
      }
```

#### Step5 :
new query log file will be creted in `storage/logs/queries.log`.

Example query logs:-
```
[2024-09-25 19:57:33] prodcution.INFO: select * from `messages` where `status` = 0 and `scchuduled_at` < 2024-09-25 19:57:33 and `campaign_id` in (select `id` from `wa_campaings` where `is_active` = 1) and `company_id` = 3 limit 200 {"time":20.79} 
[2024-09-25 19:57:33] prodcution.INFO: select * from `wa_campaings` where `wa_campaings`.`id` = 89 and `company_id` = 3 limit 1 {"time":0.99} 
[2024-09-25 19:57:33] prodcution.INFO: select * from `companies` where `companies`.`id` = 3 and `companies`.`deleted_at` is null limit 1 {"time":1.16} 
```

In logs you will get all database queries and time in milliseconds. 
