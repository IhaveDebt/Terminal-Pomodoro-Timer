use clap::Parser;
use std::{thread, time::Duration, io::{self, Write}};

#[derive(Parser)]
#[command(name = "pomodoro", about = "Simple terminal Pomodoro timer")]
struct Args {
    /// Work minutes (default 25)
    #[arg(short, long, default_value_t = 25)]
    work: u64,

    /// Short break minutes (default 5)
    #[arg(short='b', long, default_value_t = 5)]
    short_break: u64,

    /// Long break minutes (default 15)
    #[arg(short='l', long, default_value_t = 15)]
    long_break: u64,

    /// Number of pomodoro cycles before long break (default 4)
    #[arg(short='c', long, default_value_t = 4)]
    cycles: u32,
}

fn countdown(minutes: u64, label: &str) {
    let total_secs = minutes * 60;
    for elapsed in 0..=total_secs {
        let remaining = total_secs - elapsed;
        let m = remaining / 60;
        let s = remaining % 60;
        print!("\r{} – {:02}:{:02} remaining ", label, m, s);
        io::stdout().flush().ok();
        if remaining == 0 { break; }
        thread::sleep(Duration::from_secs(1));
    }
    println!("\n{} finished!", label);
}

fn main() {
    let args = Args::parse();
    println!("Pomodoro: work={}m short_break={}m long_break={}m cycles={}",
        args.work, args.short_break, args.long_break, args.cycles);

    let mut completed = 0u32;
    loop {
        completed += 1;
        println!("\nCycle {} — Work for {} minutes", completed, args.work);
        countdown(args.work, "Work");

        if completed % args.cycles == 0 {
            println!("Long break for {} minutes", args.long_break);
            countdown(args.long_break, "Long break");
        } else {
            println!("Short break for {} minutes", args.short_break);
            countdown(args.short_break, "Short break");
        }

        println!("Press ENTER to start next work session or type 'q' + ENTER to quit.");
        let mut input = String::new();
        std::io::stdin().read_line(&mut input).ok();
        if input.trim().eq_ignore_ascii_case("q") {
            println!("Exiting Pomodoro. Good job!");
            break;
        }
    }
}
