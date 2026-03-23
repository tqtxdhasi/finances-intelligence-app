# 09 – Future Enhancements (V2+)

## V2

- **Local storage + sync**: Add SQLite on device, sync when online.
- **OCR integration**: Use ML Kit to extract data from images; user can edit before saving.
- **Intelligent product normalization**: Use word embeddings to suggest categories/products.
- **Push notifications**: Reminders to add receipts, alerts for unusual spending.
- **Receipt sharing**: Export as PDF with layout.

## V3

- **Predictions**: Forecast future spending based on history.
- **Multi‑user**: Family accounts with shared expenses.
- **Receipt splitting**: Split a receipt among multiple people.
- **Currency conversion in analytics**: View charts in any currency.
- **Web dashboard**: Access receipts from a browser.

## Extensibility Points

- **OCR module**: Replace with any engine (Google Cloud Vision, Tesseract).
- **Currency API**: Swap exchangerate.host with premium service.
- **File storage**: Use different provider without changing core code.
- **Database**: Add read replicas for analytics.

## Planned Refinements

- Improve test coverage to 90%.
- Add accessibility features (VoiceOver, TalkBack).
- Internationalization (i18n) for multiple languages.
