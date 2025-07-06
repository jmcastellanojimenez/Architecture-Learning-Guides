import React, { useState } from 'react';
import { ChevronRight, ChevronDown, Package, Settings, Play, FileText } from 'lucide-react';

const MavenGuide = () => {
  const [activeSection, setActiveSection] = useState('what-is-maven');
  const [expandedConcepts, setExpandedConcepts] = useState({});

  const toggleConcept = (concept) => {
    setExpandedConcepts(prev => ({
      ...prev,
      [concept]: !prev[concept]
    }));
  };

  const sections = [
    { id: 'what-is-maven', title: 'What is Maven?', icon: Package },
    { id: 'key-concepts', title: 'Key Concepts', icon: Settings },
    { id: 'build-lifecycle', title: 'Build Lifecycle', icon: Play },
    { id: 'pom-file', title: 'POM File', icon: FileText }
  ];

  const concepts = [
    {
      id: 'dependencies',
      title: 'Dependencies',
      analogy: 'Like ingredients in a recipe',
      description: 'External libraries your project needs to function',
      example: 'If you want to work with JSON data, you add a JSON library as a dependency'
    },
    {
      id: 'repositories',
      title: 'Repositories',
      analogy: 'Like a grocery store for code',
      description: 'Online storage where Maven finds and downloads dependencies',
      example: 'Maven Central is the main "store" where most Java libraries are kept'
    },
    {
      id: 'artifacts',
      title: 'Artifacts',
      analogy: 'Like packaged products',
      description: 'The final output of your project (JAR files, WAR files, etc.)',
      example: 'Your finished app gets packaged into a JAR file that can be run anywhere'
    },
    {
      id: 'groupid',
      title: 'GroupId & ArtifactId',
      analogy: 'Like a mailing address',
      description: 'Unique identifiers for your project and its components',
      example: 'com.company.myapp identifies your specific project uniquely'
    }
  ];

  const buildPhases = [
    { phase: 'validate', description: 'Check if project is correct' },
    { phase: 'compile', description: 'Convert source code to bytecode' },
    { phase: 'test', description: 'Run automated tests' },
    { phase: 'package', description: 'Bundle code into JAR/WAR files' },
    { phase: 'install', description: 'Put package in local repository' },
    { phase: 'deploy', description: 'Share package with others' }
  ];

  return (
    <div className="max-w-4xl mx-auto p-6 bg-gray-50 min-h-screen">
      <div className="bg-white rounded-lg shadow-lg overflow-hidden">
        {/* Header */}
        <div className="bg-blue-600 text-white p-6">
          <h1 className="text-3xl font-bold mb-2">Maven Explained Simply</h1>
          <p className="text-blue-100">Your friendly guide to understanding Maven build automation</p>
        </div>

        {/* Navigation */}
        <div className="border-b border-gray-200">
          <nav className="flex">
            {sections.map((section) => {
              const Icon = section.icon;
              return (
                <button
                  key={section.id}
                  onClick={() => setActiveSection(section.id)}
                  className={`flex items-center px-6 py-4 text-sm font-medium ${
                    activeSection === section.id
                      ? 'text-blue-600 border-b-2 border-blue-600 bg-blue-50'
                      : 'text-gray-600 hover:text-blue-600 hover:bg-gray-50'
                  }`}
                >
                  <Icon className="w-4 h-4 mr-2" />
                  {section.title}
                </button>
              );
            })}
          </nav>
        </div>

        {/* Content */}
        <div className="p-6">
          {activeSection === 'what-is-maven' && (
            <div className="space-y-6">
              <div className="bg-blue-50 p-4 rounded-lg">
                <h2 className="text-xl font-semibold mb-3 text-blue-800">Think of Maven as...</h2>
                <p className="text-gray-700">A smart kitchen assistant that:</p>
                <ul className="mt-2 space-y-1 text-gray-700">
                  <li>📋 Knows what ingredients (dependencies) your recipe needs</li>
                  <li>🛒 Automatically goes shopping for those ingredients</li>
                  <li>👨‍🍳 Follows the same cooking steps every time</li>
                  <li>📦 Packages the final dish nicely for serving</li>
                </ul>
              </div>

              <div className="grid md:grid-cols-2 gap-4">
                <div className="bg-green-50 p-4 rounded-lg">
                  <h3 className="font-semibold text-green-800 mb-2">What Maven Does</h3>
                  <ul className="text-sm text-gray-700 space-y-1">
                    <li>✅ Manages project dependencies</li>
                    <li>✅ Compiles your code</li>
                    <li>✅ Runs tests automatically</li>
                    <li>✅ Packages your application</li>
                    <li>✅ Handles deployment</li>
                  </ul>
                </div>
                <div className="bg-yellow-50 p-4 rounded-lg">
                  <h3 className="font-semibold text-yellow-800 mb-2">Why Use Maven?</h3>
                  <ul className="text-sm text-gray-700 space-y-1">
                    <li>⚡ Saves time on setup</li>
                    <li>🔄 Standardizes project structure</li>
                    <li>🤝 Makes collaboration easier</li>
                    <li>🔧 Handles complex build processes</li>
                    <li>📚 Huge ecosystem of plugins</li>
                  </ul>
                </div>
              </div>
            </div>
          )}

          {activeSection === 'key-concepts' && (
            <div className="space-y-4">
              <h2 className="text-xl font-semibold mb-4">Core Concepts</h2>
              {concepts.map((concept) => (
                <div key={concept.id} className="border border-gray-200 rounded-lg">
                  <button
                    onClick={() => toggleConcept(concept.id)}
                    className="w-full px-4 py-3 text-left flex items-center justify-between hover:bg-gray-50"
                  >
                    <div className="flex items-center">
                      <span className="font-medium">{concept.title}</span>
                      <span className="ml-2 text-sm text-gray-500">- {concept.analogy}</span>
                    </div>
                    {expandedConcepts[concept.id] ? <ChevronDown className="w-4 h-4" /> : <ChevronRight className="w-4 h-4" />}
                  </button>
                  {expandedConcepts[concept.id] && (
                    <div className="px-4 pb-4 border-t border-gray-100">
                      <p className="text-gray-700 mb-2">{concept.description}</p>
                      <div className="bg-gray-50 p-3 rounded text-sm">
                        <strong>Example:</strong> {concept.example}
                      </div>
                    </div>
                  )}
                </div>
              ))}
            </div>
          )}

          {activeSection === 'build-lifecycle' && (
            <div className="space-y-6">
              <h2 className="text-xl font-semibold mb-4">Build Lifecycle</h2>
              <p className="text-gray-700 mb-4">Maven follows a predictable sequence of steps to build your project:</p>
              
              <div className="space-y-3">
                {buildPhases.map((phase, index) => (
                  <div key={phase.phase} className="flex items-center">
                    <div className="flex-shrink-0 w-8 h-8 bg-blue-500 text-white rounded-full flex items-center justify-center text-sm font-medium mr-4">
                      {index + 1}
                    </div>
                    <div className="flex-1 bg-white border border-gray-200 rounded-lg p-3">
                      <div className="font-medium text-gray-800">{phase.phase}</div>
                      <div className="text-sm text-gray-600">{phase.description}</div>
                    </div>
                  </div>
                ))}
              </div>

              <div className="bg-blue-50 p-4 rounded-lg">
                <h3 className="font-semibold text-blue-800 mb-2">Common Commands</h3>
                <div className="space-y-2 text-sm">
                  <div><code className="bg-gray-200 px-2 py-1 rounded">mvn compile</code> - Just compile the code</div>
                  <div><code className="bg-gray-200 px-2 py-1 rounded">mvn test</code> - Compile and run tests</div>
                  <div><code className="bg-gray-200 px-2 py-1 rounded">mvn package</code> - Build the complete package</div>
                  <div><code className="bg-gray-200 px-2 py-1 rounded">mvn clean install</code> - Full clean build</div>
                </div>
              </div>
            </div>
          )}

          {activeSection === 'pom-file' && (
            <div className="space-y-6">
              <h2 className="text-xl font-semibold mb-4">POM File (Project Object Model)</h2>
              <p className="text-gray-700 mb-4">The pom.xml file is like a recipe card for your project. It tells Maven everything it needs to know:</p>
              
              <div className="bg-gray-50 p-4 rounded-lg">
                <h3 className="font-semibold mb-3">Sample POM Structure</h3>
                <pre className="text-sm bg-white p-3 rounded border overflow-x-auto">
{`<?xml version="1.0" encoding="UTF-8"?>
<project>
  <!-- Who made this project? -->
  <groupId>com.mycompany</groupId>
  <artifactId>my-awesome-app</artifactId>
  <version>1.0.0</version>
  
  <!-- What do we need? -->
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
    </dependency>
  </dependencies>
</project>`}
                </pre>
              </div>

              <div className="grid md:grid-cols-2 gap-4">
                <div className="bg-green-50 p-4 rounded-lg">
                  <h3 className="font-semibold text-green-800 mb-2">Required Elements</h3>
                  <ul className="text-sm text-gray-700 space-y-1">
                    <li><strong>groupId:</strong> Your organization</li>
                    <li><strong>artifactId:</strong> Project name</li>
                    <li><strong>version:</strong> Current version</li>
                  </ul>
                </div>
                <div className="bg-blue-50 p-4 rounded-lg">
                  <h3 className="font-semibold text-blue-800 mb-2">Optional Elements</h3>
                  <ul className="text-sm text-gray-700 space-y-1">
                    <li><strong>dependencies:</strong> External libraries</li>
                    <li><strong>plugins:</strong> Build tools</li>
                    <li><strong>properties:</strong> Configuration</li>
                  </ul>
                </div>
              </div>
            </div>
          )}
        </div>
      </div>
    </div>
  );
};

export default MavenGuide;
