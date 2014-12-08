task :default => :server

desc 'Build site with Jekyll'
task :build do
  jpid = jekyll('build')

  trap_and_kill jpid
end

desc 'Start server with --auto'
task :server do
  jpid = jekyll('serve --watch --host localhost --port 4000')

  trap_and_kill jpid
end


def jekyll(opts = '')
  fork do
    sh 'jekyll ' + opts
  end
end

def trap_and_kill(*pids)
  Signal.trap("TERM") do
    pids.each do |pid|
      Process.kill("TERM", pid)
    end
  end

  Signal.trap("INT") do
    pids.each do |pid|
      Process.kill("INT", pid)
    end
  end

  Process.waitall
end
