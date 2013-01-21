---
layout: post
title: "groovy testng tests"
date: 2013-01-21 13:52
author: Kamil Podlesak
comments: true
categories: java
---


This post is not about using groovy to write unit tests (just warning for those who haven't tried it yet and still use java: it's addictive), but rather about one particular problem:
how to run them? Maven surefire plugin should, in theory, run them - there is even explicit support. Unfortunately, this plugin is (like many maven plugins) riddled with bugs and I haven't seen
single version without at least one bug in testng support. Usually, it fails to find groovy tests.

One obvious workaround is to compile groovy to `.class` files. When compiled, groovy classes are found by surefire just like java ones. This solution, however, is quite complicated and just feels wrong.

Another solution is quite obscure, but perfect: testng `@Factory` annotation.
This method is called on any testng instance that had been already found and the result (array) is then added to list of testng instances to execute. In short, this allows us to "spawn" other tests.

Less talking, more code:
{% codeblock lang:java %}
import groovy.lang.GroovyClassLoader;
import groovy.lang.Script;
import org.codehaus.groovy.control.CompilationFailedException;
import org.codehaus.groovy.runtime.DefaultGroovyMethods;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.testng.annotations.Factory;

import java.util.ArrayList;
import java.util.List;

/**
 * This is not really test, this is just runner fro groovy tests.
 * Maven Surefire plugin can, in theory, run these tests by itself; unfortunately it fails on classes
 * without default constructor and on scripts.
 */
public class GroovyRunnerTest {

    protected final GroovyClassLoader loader = new GroovyClassLoader(getClass().getClassLoader());

    @Factory()
    public Object[] createTests() throws Exception {
        System.out.println("createTests: " + loader);

        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver(loader);
        final Resource[] resources = resolver.getResources("classpath:**/*Test.groovy");
        List<Object> tests = new ArrayList<Object>(resources.length);
        for (Resource resource : resources) {
            final String text = DefaultGroovyMethods.getText(resource.getInputStream(), "UTF-8");
            try {
                final Class<?> clazz = loader.parseClass(text, resource.getFilename());
                System.out.println("groovy class: " + clazz);
                if (Script.class.isAssignableFrom(clazz)) {
                    System.out.println("skipping groovy script: " + resource);
                    continue;
                }
                final Object instance = clazz.newInstance();
                tests.add(instance);
            } catch (CompilationFailedException e) {
                System.out.println("failed to load class: " + resource);
            } catch (InstantiationException e) {
                System.out.println("failed to instantiate class: " + resource);
                System.err.println("failed to instantiate class: " + e);
                //ignore
            }

        }

        return tests.toArray(new Object[tests.size()]);
    }

}
{% endcodeblock %}

