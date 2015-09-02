require 'fileutils'
require 'rake/clean'
require 'yaml'

BOOK = YAML.load_file('config.yml')['bookname']
BOOK_PDF = BOOK+".pdf"
BOOK_EPUB = BOOK+".epub"
CONFIG_FILE = "config.yml"

def build(mode, chapter)
  sh "review-compile --target=#{mode} --footnotetext --stylesheet=style.css #{chapter} > tmp"
  mode_ext = {"html" => "html", "latex" => "tex",
              "idgxml" => "xml", "inao" => "inao"}
  FileUtils.mv "tmp", chapter.gsub(/re\z/, mode_ext[mode])
end

def build_all(mode)
  sh "review-compile --all --target=#{mode} --footnotetext --stylesheet=style.css"
end

task :default => :html_all

desc "build html (Usage: rake build re=target.re)"
task :html do
  if ENV['re'].nil?
    puts "Usage: rake build re=target.re"
    exit
  end
  build("html", ENV['re'])
end

desc "build all html"
task :html_all do
  build_all("html")
end

desc 'generate PDF and EPUB file'
task :all => [:pdf, :epub]

desc 'generate PDF file'
task :pdf => BOOK_PDF

desc 'generate EPUB file'
task :epub => BOOK_EPUB

SRC = FileList['*.md'].collect{|f| f.gsub(/\.md$/, '.re')} + [CONFIG_FILE]

file BOOK_PDF => SRC do
  FileUtils.rm_rf [BOOK_PDF, BOOK, BOOK+"-pdf"]
  sh "review-pdfmaker #{CONFIG_FILE}"
end

file BOOK_EPUB => SRC do
  FileUtils.rm_rf [BOOK_EPUB, BOOK, BOOK+"-epub"]
  sh "review-epubmaker #{CONFIG_FILE}"
end

CLEAN.include([BOOK, BOOK_PDF, BOOK_EPUB, BOOK+"-pdf", BOOK+"-epub"])

rule '.re' => ['.md'] do |t|
  sh "md2review #{t.source} > #{t.name}"
end
CLEAN.include( FileList['*.re'] )

task :printable => BOOK_PDF do
  pagecount=`pdf2ps #{BOOK_PDF} /dev/stdout | gs -sDEVICE=bbox -o /dev/null /dev/stdin 2>&1 | grep HiResBoundingBox | wc -l`.to_i
  sh "pdfbook --short-edge #{BOOK_PDF} '2,4-#{pagecount-1}'    -o printable-body.pdf"
  sh "pdfnup  --nup 2x1    #{BOOK_PDF} '#{pagecount}' cover.pdf -o printable-cover.pdf"
end
CLEAN.include( ['printable-body.pdf', 'printable-cover.pdf'] )

task :publish => BOOK_PDF do
  sh "pdfjam cover.pdf 1 #{BOOK_PDF} '2,4-'    -o #{BOOK + '-publish.pdf'}"
end
CLEAN.include( [BOOK+'-publish.pdf'] )

