require 'shellwords'

namespace :doc do
  def docset(path = "")
    File.join(*["HTTP Statuses.docset", path.split("/")].flatten.compact)
  end

  task :download do
    sh "wget -m -p -c -k --user-agent= -e robots=off --wait 1 -E -np http://httpstatus.es/"
    mkdir_p docset("Contents/Resources")
    mv "httpstatus.es", docset("Contents/Resources/Documents")
    cp "icon.png", docset
    cp "Info.plist", docset("Contents")
  end

  task :index do
    def db
      docset('Contents/Resources/docSet.dsidx')
    end

    def docdir
      docset('Contents/Resources/Documents')
    end

    def sql(*cmd)
      IO.popen("sqlite3 #{Shellwords.escape(db)}", 'w+'){|sql| cmd.each{|l| sql << l } }
    end

    def index(name, type, path)
      sql("INSERT OR IGNORE INTO searchIndex(name, type, path) VALUES ('#{name}', '#{type}', '#{path}');")
    end

    # Create the database and schema
    rm_rf db if File.exist?(db)
    sql "CREATE TABLE searchIndex(id INTEGER PRIMARY KEY, name TEXT, type TEXT, path TEXT);",
      "CREATE UNIQUE INDEX anchor ON searchIndex (name, type, path);"

    Dir.glob("#{docdir}/{1,2,3,4,5}*.html").each do |file|
      name = file.match(/(\d+)\.html/){|m| m[1] }
      index name, "Value", File.basename(file)
    end
  end

  task :generate => [:download, :index]
end

task :default => "doc:generate"
