# 引言
最近在看的书讲到了设计模式，恰好项目上重构的时候使用到了，下面具体给大家分享下，希望对大家有所帮助。

# 第一个故事
先来看下具体的代码：

##### ExportExcelJob:
```ruby
def perform(session_id, report_id)
  report = AmsReport.find report_id
  report_data = report.dto

  excel = create_excel_with_regular report_data[:regular]
  append_outcomes(excel, report_data[:outcome])
  append_section_comments(excel, report_data[:section_comment])

  file = Tempfile.new(%w(excel_report .xlsx))
  excel.write(file.path)
  excel_file = LT::Uploader.new(session_id, file.path)

  excel_file.upload do
    fileInfo = excel_file.into_file_manager
    if fileInfo
      file_record = AmsReportFile.where(report_id: report_id).last
      file_id = fileInfo['file_id']
      file_record.file_id = file_id;
      file_record.status = 'done';
      file_record.save
    end
  end
  file.close
  file.unlink
end
```
##### ExportWordJob:
```ruby
def perform(session_id, report_id)
  report = AmsReport.find(report_id).reports_dto
  word = WordBuilder.new 'session_id', 'word_group_by_section', {:report  => report }
  word.build
end
```

对比可以发现`ExportExcelJob`本身处理了太多不应该处理的逻辑(创建临时文件，上传文件，关闭临时文件)，它的职责应该只是将必要参数传递给`Builder`并调用`build`方法就可以了，正如`ExportWordJob`一样。所以我们将不属于它的逻辑移动到`ExcelBuilder`中去, 我们不妨先看一下`WordBuilder`是怎么做的：

```ruby
class WordBuilder < Builder

  def build
    file = Tempfile.new(%w(word_report .docx))
    html = render_html
    file.write(html)
    file.rewind
    %x( pandoc -o #{file.path} -t docx -f html #{file.path})
    upload file
  end
end
```
我们可以看到它只包含了一个方法就是`build`, 仔细看细节我们可以发下它使用了父类的`render_html`, `upload`方法将report内容写进文件并上传。我们发现原来我们已经实现过了文件上传的方法，所以对于`ExcelBuilder`如果继承了`Builder`也可以直接调用父类的`upload`方法，很好我们已经去掉了一个重复方法了，可以庆祝一下了。

```ruby
class ExcelBuilder < Builder

  def build
    file = Tempfile.new(%w(excel_report .xlsx))
    @workbook = RubyXL::Workbook.new
    @headers = {}
    create_excel_with_regular
    append_outcomes
    append_section_comments
    @workbook.write file.path
    upload file
  end
```
看起来我们似乎已经取得了不错的成果，可仔细看的话会发现`ExcelBuilder`与`WordBuilder`还是有很多相似的代码，我们很容易发现同样可以将创建临时文件的方法同样放到父类里，并在子类里面直接使用:

```ruby
class Builder
  def initialize(session_id, report_id)
    @session_id, @report_id = session_id, report_id
  end

  def create_tmp_file(file_name)
    ext_name = file_name.split('.').last
    base_name = file_name.gsub(".#{ext_name}", '')
    Tempfile.new([base_name, ".#{ext_name}"])
  end

  def close_tmp_file file
    file.close
    file.unlink
  end

  def upload
    file_record = AmsReportFile.where(report_id: @report_id).last
    update_status(file_record, 'uploading')
    c1 = LT::Uploader.new(@session_id, @file.path)
    c1.upload do |status|
      if status
        c1.into_file_manager do |file_info|
          if file_info
            update_status(file_record, 'done', file_info['file_id'])
          else
            update_status(file_record, 'failed')
          end
        end
      else
        update_status(file_record, 'failed')
      end
    end
  end

  def update_status(file_record, status, file_id = nil)
    file_record.file_id = file_id
    file_record.status = status
    file_record.save
  end
end
```
现在父类`Builder`中包含了`initialize`（初始化以接收生成report所需数据）, `create_tmp_file`, `close_tmp_file`, `upload`, `update_status`（用以更新report上传状态）五个公用方法，子类中均可直接使用。

```ruby
class WordBuilder < Builder

  def build
    file = create_tmp_file('word_report.docx')
    html = render_html
    file.write(html)
    file.rewind
    %x( pandoc -o #{file.path} -t docx -f html #{file.path})
    upload file
    close_tmp_file file
  end
end

class ExcelBuilder < Builder

  def build
    file = create_tmp_file(`excel_report.xlsx`)
    @workbook = RubyXL::Workbook.new
    @headers = {}
    create_excel_with_regular
    append_outcomes
    append_section_comments
    @workbook.write file.path
    upload file
    close_tmp_file file
  end

  private
  ...
end
```
好的这时我们代码似乎已经重构完成了，我们已经去掉了"所有"重复的代码了。真的如此吗？ 仔细观察，我们可以发现，其实我们忽略掉了一个很重要的部分，那就是流程，这两个`Builder`所包含的流程其实也是相同的，创建临时文件 -> **写入数据** -> 上传临时文件 -> 关闭临时文件。所以我们可以将整个流程放在父类中，而子类只需实现不同的一步就可以了，修改后的代码如下：

```ruby
class Builder
  def initialize(session_id, report_id)
    @session_id, @report_id = session_id, report_id
  end

  def build(params, file)
  end

  def generate_report(params, file_name)
    @file = create_tmp_file(file_name)
    build params, @file
    upload
    close_tmp_file
  end

  #others methods
  ...
end

class WordBuilder < Builder
  def build(params, file)
    template = TemplateBuilder.new.render params[:template], params
    file.write(template)
    file.rewind
    %x( pandoc -o "#{file.path}" -t docx -f html "#{file.path}")
  end
end

class ExcelBuilder < Builder
  def build(params, file)
    report_data = params[:report]
    @workbook = RubyXL::Workbook.new
    @headers = {}
    create_excel_with_regular
    append_outcomes
    append_section_comments
    @workbook.write file.path
  end

  private
  ...
end

class ExportWordJob < ActiveJob::Base
  queue_as :word_job_queue

  def perform(session_id, report_id)
    report = AmsReport.find(report_id).reports_dto
    word = WordBuilder.new session_id, report_id
    word.generate_report({:template => 'word_group_by_section', :report  => report }, 'word_report.docx')
  end
end

class ExportExcelJob < ActiveJob::Base
  queue_as :excel_job_queue

  def perform(session_id, report_id)
    report = AmsReport.find(report_id).dto
    excel_report = ExcelBuilder.new session_id, report_id
    excel_report.generate_report({:report  => report }, 'excel_report.xlsx')
  end
end
```
如此我们只需在子类中实现**写入数据**一步，而在父类中定义生成report的具体流程，调用的地方只需调用统一的`generate_report`的方法就可以了。好的看到这里如果你都理解了的话，那么恭喜你，你已经学会了一个设计模式--[Template Method Pattern](https://en.wikipedia.org/wiki/Template_method_pattern)。

`PS：其实这里还可以继续重构，我们可以发现两个Job唯一的区别就是他们使用的Builder不同，如果我们将Builder作为参数传入，那么我们只需要一个Job就可以了，这里为了明确区分两种Report我们并没有这么做。`

##另一个思路：
当我们将公用的方法移动到父类`Builder`中后，其实我们还有另一个方法去处理**不同**的**写入数据**这一步，具体如下：

```ruby
class DataProcessor
    def process params, file
    end
end

class WordDataProcessor < DataProcessor
  def process params, file
    template = TemplateBuilder.new.render params[:template], params
    file.write(template)
    file.rewind
    %x( pandoc -o "#{file.path}" -t docx -f html "#{file.path}")
  end
end

class ExcelDataProcessor < DataProcessor
  def process params, file
    report_data = params[:report]
    @workbook = RubyXL::Workbook.new
    @headers = {}
    create_excel_with_regular
    append_outcomes
    append_section_comments
    @workbook.write file.path
  end

  private
  ...

end

class Builder
  def initialize(session_id, report_id, data_processor)
    @session_id, @report_id , @data_processor= session_id, report_id, data_processor
  end

  def generate_report(params, file_name)
    @file = create_tmp_file(file_name)
    data_processor.process params, @file
    upload
    close_tmp_file
  end

  #others methods
  ...

end

class ExportWordJob < ActiveJob::Base
  queue_as :word_job_queue

  def perform(session_id, report_id)
    report = AmsReport.find(report_id).reports_dto
    word_data_processor = WordDataProcessor.new
    word = Builder.new session_id, report_id, word_processor
    word.generate_report({:template => 'word_group_by_section', :report  => report }, 'word_report.docx')
  end
end

class ExportExcelJob < ActiveJob::Base
  queue_as :excel_job_queue

  def perform(session_id, report_id)
    report = AmsReport.find(report_id).dto
    excel_data_processor = ExcelDataProcessor.new
    excel_report = Builder.new session_id, report_id
    excel_report.generate_report({:report  => report }, 'excel_report.xlsx')
  end
end
```
如果你也理解了这一种方法，那么恭喜你又学会了一种设计模式--[Strategy Pattern](https://en.wikipedia.org/wiki/Strategy_pattern)。那么他们的区别是什么呢？我们可以发现，`Template Method Pattern`主要利用了继承，它的结构相对比较简单，而`Strategy Pattern`引入了一个新的接口，相对层次复杂一些，那么你觉的哪一种更好呢?

`PS：当你思考哪一种更不错的时候是否想到了一个的设计原则--多用组合少用继承，而策略模式与模板方法模式恰恰体现了这里的组合与继承，那么到底该如何抉择呢？`

今天给大家结合我们的项目介绍了一下模板方法模式和策略模式，希望能引起大家的一些思考，后面还会给大家介绍另外一个最近项目上用到的设计模式，且听下回分解。

