using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Xml.Linq;

public class XmlDifference
{
    public static void Main(string[] args)
    {
        if (args.Length != 2)
        {
            Console.WriteLine("Usage: XmlDifference <file1.xml> <file2.xml>");
            return;
        }

        string file1 = args[0];
        string file2 = args[1];

        try
        {
            XDocument doc1 = XDocument.Load(file1);
            XDocument doc2 = XDocument.Load(file2);

            List<string> differences = FindXmlDifferences(doc1, doc2);

            if (differences.Count == 0)
            {
                Console.WriteLine("XML files are identical (ignoring order).");
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
        catch (FileNotFoundException ex)
        {
            Console.WriteLine($"Error: File not found: {ex.FileName}");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"An error occurred: {ex.Message}");
        }
    }

    public static List<string> FindXmlDifferences(XDocument doc1, XDocument doc2)
    {
        List<string> differences = new List<string>();

        CompareElements(doc1.Root, doc2.Root, differences);

        return differences;
    }

    private static void CompareElements(XElement element1, XElement element2, List<string> differences)
    {
        if (element1.Name != element2.Name)
        {
            differences.Add($"Element name mismatch: {element1.Name} vs {element2.Name}");
            return;
        }

        // Compare attributes
        CompareAttributes(element1.Attributes(), element2.Attributes(), differences, element1.Name.ToString());

        // Compare values
        if (element1.HasElements == false && element2.HasElements == false && element1.Value != element2.Value)
        {
            differences.Add($"Value mismatch in {element1.Name}: '{element1.Value}' vs '{element2.Value}'");
        }

        // Compare child elements (ignoring order)
        CompareChildElements(element1.Elements(), element2.Elements(), differences);
    }

    private static void CompareAttributes(IEnumerable<XAttribute> attributes1, IEnumerable<XAttribute> attributes2, List<string> differences, string elementName)
    {
        var dict1 = attributes1.ToDictionary(a => a.Name.ToString(), a => a.Value);
        var dict2 = attributes2.ToDictionary(a => a.Name.ToString(), a => a.Value);

        var allKeys = dict1.Keys.Union(dict2.Keys).Distinct();

        foreach (var key in allKeys)
        {
            if (!dict1.ContainsKey(key))
            {
                differences.Add($"Attribute '{key}' missing in {elementName}.");
            }
            else if (!dict2.ContainsKey(key))
            {
                differences.Add($"Attribute '{key}' missing in {elementName}.");
            }
            else if (dict1[key] != dict2[key])
            {
                differences.Add($"Attribute '{key}' mismatch in {elementName}: '{dict1[key]}' vs '{dict2[key]}'");
            }
        }
    }

    private static void CompareChildElements(IEnumerable<XElement> children1, IEnumerable<XElement> children2, List<string> differences)
    {
        var list1 = children1.ToList();
        var list2 = children2.ToList();

        while (list1.Count > 0)
        {
            var element1 = list1[0];
            list1.RemoveAt(0);

            var matchingElement = list2.FirstOrDefault(e => e.Name == element1.Name && AreElementsSimilar(element1,e));

            if (matchingElement != null)
            {
                list2.Remove(matchingElement);
                CompareElements(element1, matchingElement, differences);
            }
            else
            {
                differences.Add($"Element '{element1.Name}' missing or different in the second document.");
            }
        }

        foreach(var remainingElement in list2)
        {
            differences.Add($"Element '{remainingElement.Name}' missing or different in the first document.");
        }
    }

    private static bool AreElementsSimilar(XElement element1, XElement element2)
    {
        if(element1.Name != element2.Name) return false;

        if(!element1.Attributes().SequenceEqual(element2.Attributes())) return false;

        if (element1.HasElements == false && element2.HasElements == false && element1.Value != element2.Value) return false;

        return true;
    }
}
