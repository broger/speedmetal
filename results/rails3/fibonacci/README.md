# About the benchmark

The Fibonacci benchmark calculates the Nth Fibonacci number on every
request, where N is the value generated by basho_bench's
key_generator. See the fibonacci.config entries in the README.md under
each test but so far they all generate values from 0 to 20.

This test completely saturates the CPU of the server instance. The
goal is to demonstrate how each server behaves under a high CPU load
with a varying number of concurrent clients.

# Install Gems
    jruby -S gem install bundler
    cd speedmetal/apps/rails3/fibonacci
    jruby -S bundle install
    gem install bundler
    bundle install


# TorqueBox Setup

## Create TorqueBox deployment descriptor
    rm -f $JBOSS_HOME/server/default/deploy/*-knob.yml
    cat << EOF > $JBOSS_HOME/server/default/deploy/fibonacci-knob.yml
    ---
    application:
      root: /home/ec2-user/speedmetal/apps/rails3/fibonacci/
      env: production
    web:
      context: /
    EOF
## Start TorqueBox
    $JBOSS_HOME/bin/run.sh -b 0.0.0.0


# Passenger Setup

## Copy app to /tmp
Passenger needs the nginx workers that run as the 'nobody' user
to have access to the application.
    rm -rf /tmp/fibonacci/
    cp -r speedmetal/apps/rails3/fibonacci/ /tmp/

## Start Passenger
    cd /tmp/fibonacci/
    passenger start -p 8080 -e production --max-pool-size N


# Thin Setup

## Start Thin
    cd speedmetal/apps/rails3/fibonacci/
    thin -p 8080 -e production start


# Trinidad Setup

## Start Trinidad
    cd speedmetal/apps/rails3/fibonacci/
    jruby -S trinidad -r -p 8080 -e production -t


# Unicorn Setup

## Write Config File
    cat << EOF > /tmp/unicorn.rb
    worker_processes N
    preload_app true
    timeout 30
    listen 8080, :backlog => 2048
    EOF

## Start Unicorn
    cd speedmetal/apps/rails3/fibonacci/
    unicorn_rails -E production -c /tmp/unicorn.rb
