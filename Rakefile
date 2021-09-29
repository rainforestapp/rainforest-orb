# frozen_string_literal: true

VERSION = Gem::Version.new('3.0.2')

def components
  # changes [1, 3] to [1, 3, 0]
  3.times.zip(VERSION.canonical_segments).map { |_i, s| s.to_i }
end

def update(*version)
  old_version = VERSION.version
  new_version = version.map(&:to_s).join('.')
  `git grep -z -l -F #{old_version} | xargs -0 ruby -pi -e "gsub(/#{old_version}/, '#{new_version}')"`
end

namespace :update do
  task :major do
    update(components[0] + 1, 0, 0)
  end

  task :minor do
    update(components[0], components[1] + 1, 0)
  end

  task :patch do
    update(components[0], components[1], components[2] + 1)
  end
end
