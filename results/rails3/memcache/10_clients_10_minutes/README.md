# Running the benchmark
    cat << EOF > memcache.config
    {mode, max}.
    {duration, 10}.
    {concurrent, 10}.
    {driver, basho_bench_driver_http}.
    {code_paths, ["deps/stats",
                  "deps/ibrowse"]}.
    {key_generator, {int_to_str, {uniform_int, 10000}}}.
    {value_generator, {bin_to_base64, {fixed_bin, 100}}}.
    {operations, [{get, 4},{update, 2}]}.
    {http_ips, ["10.192.65.140"]}.
    {http_port, 8080}.
    {http_path, ""}.
    EOF

    ./basho_bench memcache.config

    /usr/bin/Rscript --vanilla priv/summary.r -i tests/current

Replace the `http_raw_ips` value with the IP of your server
instance. Run the benchmark once for every server you want to test,
ensuring that only one server is running on the server instance at a
time.
