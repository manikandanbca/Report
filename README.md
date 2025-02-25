using System;
using System.Collections.Generic;
using System.Linq;
using System.Xml.Linq;

public class XmlComparer
{
    public static void Main(string[] args)
    {
        string file1Path = "file1.xml"; // Replace with your file paths
        string file2Path = "file2.xml";

        try
        {
            XDocument doc1 = XDocument.Load(file1Path);
            XDocument doc2 = XDocument.Load(file2Path);

            List<string> differences = CompareXml(doc1.Root, doc2.Root);

            if (differences.Any())
            {
                Console.WriteLine("Differences found:");
                foreach (string diff in differences)
                {
                    Console.WriteLine(diff);
                }
            }
            else
            {
                Console.WriteLine("No differences found.");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("Error: " + ex.Message);
        }
    }


    public static List<string> CompareXml(XElement element1, XElement element2)
    {
        List<string> differences = new List<string>();

        if (element1.Name != element2.Name)
        {
            differences.Add($"Element name mismatch: {element1.Name} vs {element2.Name}");
            return differences; // If names differ, no need to compare children
        }

        // Compare attributes (ignoring order)
        var attributes1 = element1.Attributes().OrderBy(a => a.Name.LocalName).ToList();
        var attributes2 = element2.Attributes().OrderBy(a => a.Name.LocalName).ToList();

        if (attributes1.Count != attributes2.Count)
        {
            differences.Add($"Attribute count mismatch in element {element1.Name}");
        }
        else
        {
            for (int i = 0; i < attributes1.Count; i++)
            {
                if (attributes1[i].Name != attributes2[i].Name || attributes1[i].Value != attributes2[i].Value)
                {
                    differences.Add($"Attribute mismatch in element {element1.Name}: {attributes1[i].Name} = {attributes1[i].Value} vs {attributes2[i].Name} = {attributes2[i].Value}");
                }
            }
        }



        // Compare child elements (ignoring order)
        var children1 = element1.Elements().OrderBy(e => e.Name.LocalName).ToList(); // Order matters for counting
        var children2 = element2.Elements().OrderBy(e => e.Name.LocalName).ToList();

        if (children1.Count != children2.Count)
        {
            differences.Add($"Child element count mismatch in element {element1.Name}");
        }
        else
        {
            for (int i = 0; i < children1.Count; i++)
            {
                differences.AddRange(CompareXml(children1[i], children2[i]));
            }
        }

        // Compare values if no children (leaf nodes)
        if (!children1.Any() && !children2.Any() && element1.Value != element2.Value)
        {
            differences.Add($"Value mismatch in element {element1.Name}: {element1.Value} vs {element2.Value}");
        }

        return differences;
    }
}
