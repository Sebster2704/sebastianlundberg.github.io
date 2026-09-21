const cheerio = require("cheerio");
const fs = require("fs");

const url = "https://finera.se/lan/privatlan";

async function scrape() {
  const response = await fetch(url, {
    headers: {
      "User-Agent": "Mozilla/5.0"
    }
  });

  const html = await response.text();
  const $ = cheerio.load(html);

  const lenders = [];

  $("tr").each((index, row) => {
    const name = $(row)
      .find(".LenderTable-module__oHvHva__name")
      .text()
      .trim();

    if (!name) return;

    const cells = $(row).find("td");

    const nominalRate = $(cells[0]).text().trim();
    const effectiveRate = $(cells[1]).find(".figure").text().trim();
    const amount = $(cells[2]).text().trim();
    const term = $(cells[3]).text().trim();

    lenders.push(
      `${name} | ${nominalRate} | ${effectiveRate} | ${amount} | ${term}`
    );
  });

  fs.writeFileSync(
    "data.txt",
    "Långivare | Nominell ränta | Effektiv ränta | Belopp | Löptid\n" +
      lenders.join("\n"),
    "utf8"
  );

  console.log(`Klart! Hittade ${lenders.length} långivare.`);
}

scrape();