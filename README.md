Add to your cargo.toml:

~~~
[features]
default = ["profile"]
profile = ["screeps-timing", "screeps-timing-annotate"]

[dependencies]
screeps-timing = { git = "https://github.com/shanemadden/screeps-timing", optional = true }
screeps-timing-annotate = { git = "https://github.com/shanemadden/screeps-timing-annotate", optional = true }
~~~

Minimum setup for timing a main loop tick and dumping it to console.

~~~
fn main_loop() {
    #[cfg(feature = "profile")]
    {
        screeps_timing::start_trace(Box::new(|| (screeps::game::cpu::get_used() * 1000.0) as u64));
    }
    
    game_loop::tick();

    #[cfg(feature = "profile")]
    {
        let trace = screeps_timing::stop_trace();

        info!("{}", trace.encode_pprof_base64());
    }   
}
~~~

Annotating a function:

~~~
#[cfg_attr(feature = "profile", screeps_timing_annotate::timing)]
fn test() {
}
~~~

Annotating a module:

~~~
#[cfg_attr(feature = "profile", screeps_timing_annotate::timing)]
mod game_loop {
    pub fn tick() {
    }
}
~~~

Annotating an entire impl:

~~~
struct Foo;

#[cfg_attr(feature = "profile", screeps_timing_annotate::timing)]
impl Foo {
    pub fn test() {
    }
}
~~~

Annotating a trait impl:

~~~
struct Foo;

#[cfg_attr(feature = "profile", screeps_timing_annotate::timing)]
impl Into<u32> for Foo {
    pub fn into(&self) -> u32 {
        0
    }
}
~~~

See the screeps-timing README for optional memory profiling: a tracking allocator that attributes heap allocations to the annotated spans, and a snapshot of the JavaScript heap statistics.

* Copy the base64 output from the console into a file, e.g. `profile.b64`.
* Decode it back to binary: `base64 -d profile.b64 > profile.pb`
* Open it with [pprof](https://github.com/google/pprof): `go tool pprof -http=: profile.pb` (or `pprof -http=: profile.pb` with the standalone tool) for flame graphs, top lists, and call graphs. [speedscope](https://www.speedscope.app/) can also open pprof files.