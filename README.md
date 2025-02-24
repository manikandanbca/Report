# Report
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Xml.Linq;

public class XmlDiff
{
    public static void Main(string[] args)
    {
        if (args.Length != 2)
        {
            Console.WriteLine("Usage: XmlDiff <file1.xml> <file2.xml>");
            return;
        }

        string file1 = args[0];
        string file2 = args[1];

        if (!File.Exists(file1) || !File.Exists(file2))
        {
            Console.WriteLine("One or both files do not exist.");
            return;
        }

        try
        {
            XDocument doc1 = XDocument.Load(file1);
            XDocument doc2 = XDocument.Load(file2);

            List<string> differences = FindDifferences(doc1, doc2);

            if (differences.Count == 0)
            {
                Console.WriteLine("No differences found.");
            }
            else
            {
                Console.WriteLine("Differences found:");
                foreach (string diff in differences)
                {
                    Console.WriteLine(diff);
                }
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error: {ex.Message}");
        }
    }

    public static List<string> FindDifferences(XDocument doc1, XDocument doc2)
    {
        List<string> differences = new List<string>();

        CompareElements(doc1.Root, doc2.Root, "", differences);

        return differences;
    }

    private static void CompareElements(XElement element1, XElement element2, string path, List<string> differences)
    {
        if (element1 == null && element2 == null)
        {
            return; // Both elements are null, no difference.
        }

        if (element1 == null || element2 == null)
        {
            differences.Add($"Element mismatch at {path}: {(element1 == null ? "Missing in file 1" : "Missing in file 2")}");
            return;
        }

        if (element1.Name != element2.Name)
        {
            differences.Add($"Element name mismatch at {path}: {element1.Name} vs {element2.Name}");
            return;
        }

        if (element1.Value != element2.Value)
        {
            differences.Add($"Value mismatch at {path}/{element1.Name}: '{element1.Value}' vs '{element2.Value}'");
        }

        // Compare attributes
        var attributes1 = element1.Attributes().ToDictionary(a => a.Name.ToString(), a => a.Value);
        var attributes2 = element2.Attributes().ToDictionary(a => a.Name.ToString(), a => a.Value);

        foreach (var attr1 in attributes1)
        {
            if (!attributes2.ContainsKey(attr1.Key))
            {
                differences.Add($"Attribute '{attr1.Key}' missing in file 2 at {path}/{element1.Name}");
            }
            else if (attributes2[attr1.Key] != attr1.Value)
            {
                differences.Add($"Attribute value mismatch at {path}/{element1.Name}['{attr1.Key}']: '{attr1.Value}' vs '{attributes2[attr1.Key]}'");
            }
        }

        foreach(var attr2 in attributes2)
        {
            if(!attributes1.ContainsKey(attr2.Key))
            {
                differences.Add($"Attribute '{attr2.Key}' missing in file 1 at {path}/{element2.Name}");
            }
        }

        // Compare child elements recursively
        var children1 = element1.Elements().ToList();
        var children2 = element2.Elements().ToList();

        int maxChildren = Math.Max(children1.Count, children2.Count);

        for (int i = 0; i < maxChildren; i++)
        {
            XElement child1 = i < children1.Count ? children1[i] : null;
            XElement child2 = i < children2.Count ? children2[i] : null;

            CompareElements(child1, child2, $"{path}/{element1.Name}", differences);
        }
    }
}
