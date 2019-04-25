## Page Object Duplicates and Orphans

### Assumptions:

* Page objects are in a locators.yml file
* Locators file is one locator per line
* Locators file format is name : 'locator'
* Specs are called *_spec.rb and can be in subdirectories

    before :each do
      load_locators
    end

    it 'does not have duplicates' do
      if @keys.uniq.count != @keys.count
        dupe_keys = @keys.select{|n| @keys.count(n) > 1}.uniq
        dupe_keys.each do |key|
          @pairs.each do |pair|
            p "#{key.to_sym} : #{pair[key]}" if pair[key]
          end
        end
      end

      expect(@keys.uniq.count).to (eq @keys.count),
        lambda {"Duplicate page object keys found! #{dupe_keys}"}

    end

    it "uses all its locator keys" do
      files = Dir.glob("**/*_spec.rb")
      unused_keys = []
      @keys.each do |key|
        @key_used = false
        files.each {|file| search_file_for_key(file, key) }
        unused_keys << key unless @key_used
      end
      unused_keys_exist = unused_keys.size > 0

      expect(unused_keys_exist).not_to be,
        lambda {"Failure - orphan page object identifiers exist #{unused_keys}"}

    end

    def load_locators
      locators_file = File.open('locators.yml')
      @pairs = []
      @keys = []

      locators_file.each_line do |line|
        words = line.split(': ')
        @pairs << {words[0] => words[1]}
        @keys << words[0]
      end
      locators_file.close
    end

