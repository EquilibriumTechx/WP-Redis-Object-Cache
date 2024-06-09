<?php
// Add a custom cron schedule
add_filter('cron_schedules', 'custom_schedule_flush_cache');
function custom_schedule_flush_cache($schedules) {
    $schedules['redis_clear_flush_12hr'] = array(
        'interval' => 43200, // 12 hours in seconds
        'display' => __('Every 12 Hours', 'textdomain')
    );
    return $schedules;
}

// Schedule the cache flush if it's not already scheduled
function schedule_redis_flush_cache_event() {
    if (!wp_next_scheduled('redis_flush_cache_event')) {
        wp_schedule_event(time(), 'redis_clear_flush_12hr', 'redis_flush_cache_event');
    }
}
add_action('init', 'schedule_redis_flush_cache_event');

// Hook into the scheduled event to flush the cache
add_action('redis_flush_cache_event', 'flush_cache_function');
function flush_cache_function() {
    // Flush cache
    wp_cache_flush();

    // Send email notification
    $to = 'admin@yoursite.my'; //add your own email to get the notification as required
    $subject = 'Redis Cache Flushed';
    $message = 'The Redis cache has been flushed at ' . date('Y-m-d H:i:s');
    $headers = array('Content-Type: text/html; charset=UTF-8');

    wp_mail($to, $subject, $message, $headers);

    // Logging for debugging
    error_log('Cache flushed and email sent at ' . date('Y-m-d H:i:s'));

    // Store log in the database
    $log = get_option('redis_cache_flush_log', array());
    $log[] = 'Cache flushed at ' . date('Y-m-d H:i:s');
    update_option('redis_cache_flush_log', $log);
}

// Clear the scheduled event manually and log it
function clear_redis_flush_cache_event() {
    $timestamp = wp_next_scheduled('redis_flush_cache_event');
    if ($timestamp) {
        wp_unschedule_event($timestamp, 'redis_flush_cache_event');
    }

    // Manually flush cache and log it
    manual_flush_cache();
}

// Separate function to manually flush cache and log it
function manual_flush_cache() {
    // Flush cache
    wp_cache_flush();

    // Log manual clear event
    $log = get_option('redis_cache_flush_log', array());
    $log[] = 'Cache manually cleared and flushed at ' . date('Y-m-d H:i:s');
    update_option('redis_cache_flush_log', $log);
}

// Clear all cache flush logs
function clear_cache_flush_logs() {
    update_option('redis_cache_flush_log', array());
    wp_send_json_success('Logs cleared');
}
add_action('wp_ajax_clear_cache_flush_logs', 'clear_cache_flush_logs');

// Fetch cache flush log via Ajax
add_action('wp_ajax_fetch_cache_flush_log', 'fetch_cache_flush_log');
function fetch_cache_flush_log() {
    $log = get_option('redis_cache_flush_log', array());
    wp_send_json_success($log);
}

// Add admin page to manually clear cache and view log
add_action('admin_menu', 'custom_cache_clear_menu');
function custom_cache_clear_menu() {
    add_management_page(
        'Clear Redis Cache Event',
        'Clear Redis Cache Event',
        'manage_options',
        'clear-redis-cache-event',
        'clear_redis_cache_event_page'
    );
}

function clear_redis_cache_event_page() {
    ?>
    <div class="wrap">
        <h1>Clear Redis Cache Event</h1>
        <form method="post" action="">
            <?php
            if (isset($_POST['clear_event'])) {
                clear_redis_flush_cache_event();
                echo '<div class="updated"><p>Redis cache event cleared.</p></div>';
            }
            ?>
            <p>Click the button below to clear the scheduled Redis cache flush event.</p>
            <p><input type="submit" name="clear_event" class="button-primary" value="Clear Event"></p>
        </form>

        <h2>Cache Flush Log</h2>
        <div id="cache-flush-log">
            <p>Loading log...</p>
        </div>
        <button id="clear-logs" class="button-secondary">Clear Logs</button>

        <script type="text/javascript">
            document.addEventListener('DOMContentLoaded', function() {
                function fetchLog() {
                    var xhr = new XMLHttpRequest();
                    xhr.open('POST', '<?php echo admin_url('admin-ajax.php'); ?>');
                    xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
                    xhr.onload = function() {
                        if (xhr.status === 200) {
                            var response = JSON.parse(xhr.responseText);
                            if (response.success) {
                                var log = response.data;
                                var logContainer = document.getElementById('cache-flush-log');
                                logContainer.innerHTML = '';
                                if (log.length > 0) {
                                    var logList = document.createElement('ul');
                                    log.forEach(function(entry) {
                                        var listItem = document.createElement('li');
                                        listItem.textContent = entry;
                                        logList.appendChild(listItem);
                                    });
                                    logContainer.appendChild(logList);
                                } else {
                                    logContainer.textContent = 'No cache flush events logged.';
                                }
                            }
                        }
                    };
                    xhr.send('action=fetch_cache_flush_log');
                }

                // Fetch log on page load
                fetchLog();

                // Set interval to fetch log every 30 seconds
                setInterval(fetchLog, 30000);

                // Clear logs button click handler
                document.getElementById('clear-logs').addEventListener('click', function() {
                    var xhr = new XMLHttpRequest();
                    xhr.open('POST', '<?php echo admin_url('admin-ajax.php'); ?>');
                    xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
                    xhr.onload = function() {
                        if (xhr.status === 200) {
                            var response = JSON.parse(xhr.responseText);
                            if (response.success) {
                                fetchLog();
                            }
                        }
                    };
                    xhr.send('action=clear_cache_flush_logs');
                });
            });
        </script>
    </div>
    <?php
}
?>
