# Refactor Example

```ruby
# app/services/doc_gen/estate_documents/builder.rb
module DocGen
  module EstateDocuments
    module Builder
      ERROR_PREFIX = '[DocGen::EstateDocuments::Builder]'

      def build_estate_document(args)
        validate_params(args)
        create_estate_document(args)
      end

      private

      def validate_params(args)
        raise NilParamError.new(:kind, args) if args[:kind].nil?
      end

      def create_estate_document(args); end

      # Specific named error classes at the class/module level will help us have
      # more expressiveness and allows for better debugging
      class NilParamError < StandardError
        def initialize(param, args)
          super "#{ERROR_PREFIX}: #{param} not provided in params: #{args}"
        end
      end
    end
  end
end

# app/services/doc_gen/estate_documents/healthcare_directive.rb
module DocGen
  module EstateDocuments
    class HealthcareDirective
      extend DocGen::EstateDocuments::Builder

      def self.create(args)
        build_estate_document(
          **args,
          kind: :healthcare_directive,
          doc_name: 'Healthcare Directive'
        )
      end
    end
  end
end

# app/services/doc_gen/package/basic_trust.rb
module Package
  class BasicTrust
    include Helpers
    # Eventually, all computed values from questionnaire answers should be moved out of
    # BasicTrust and to the Builder. Currently, they are spread out between instance variables
    # in this file and the Helpers module. Ideally, we would have one single interface to access
    # everything questionnaire related
    include Builder

    def process!
      set_common_package_props(
        # ...
      )

      create_client_pourover_will { |doc| create_document_associations(doc) }
    end
  end
end

# app/services/doc_gen/package/builder.rb
module DocGen
  module Package
    # The idea here is to provide all possible build options for a package, and the
    # package itself will choose from those options
    module Builder
      def set_common_package_props(args)
        @estate = args[:estate]
        # ...
      end

      def create_client_pourover_will
        yield(DocGen::EstateDocuments::PouroverWill.create(client_document_props))
      end
    end
  end
end
```
