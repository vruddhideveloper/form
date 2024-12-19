
import csv
import os
import numpy as np
from datetime import datetime

class LatencyProcessor:
    def __init__(self, output_dir):
        self.output_dir = output_dir
        self.csv_info = {
            'request_count': {
                'filename': 'request_count.csv',
                'headers': ['Date', 'Instance Name', 'Total Requests']
            },
            'lc_latency': {
                'filename': 'LC_latency.csv',
                'headers': ['Date', 'Instance Name', 'Min', 'Mean', 'Median', 'Max'] + 
                          [f'{i}%' for i in [10, 20, 30, 40, 50, 60, 70, 80, 90, 95, 96, 97, 98, 99]]
            },
            'overall_latency': {
                'filename': 'Overall_latency.csv',
                'headers': ['Date', 'Instance Name', 'Min', 'Mean', 'Median', 'Max'] + 
                          [f'{i}%' for i in [10, 20, 30, 40, 50, 60, 70, 80, 90, 95, 96, 97, 98, 99]]
            }
        }

    def create_initial_files(self):
        """Create initial CSV files with headers"""
        for csv_info in self.csv_info.values():
            filepath = os.path.join(self.output_dir, csv_info['filename'])
            if not os.path.exists(filepath):
                with open(filepath, 'w', newline='') as f:
                    writer = csv.writer(f)
                    writer.writerow(csv_info['headers'])

    def save_stats(self, latency_data, logname, description, instance_name, output_func=print):
        """Process and save latency statistics"""
        if len(latency_data) == 0:
            print(f'No request-response in the {logname}')
            return

        try:
            arr = np.array(latency_data)
            current_date = datetime.now().strftime('%Y-%m-%d')
            
            # Calculate basic statistics
            latency = []
            latency.extend([
                np.min(arr),
                np.mean(arr),
                np.median(arr),
                np.max(arr)
            ])

            # Calculate percentiles
            percentages = [10, 20, 30, 40, 50, 60, 70, 80, 90, 95, 96, 97, 98, 99]
            for p in percentages:
                latency_value = np.percentile(arr, p)
                latency.append(latency_value)

            # Get request count
            req_count = len(arr)

            # Append data to respective CSV files
            if description == 'CIT internal: Response out - Request in':
                self.append_data_to_csv(
                    self.csv_info['lc_latency']['filename'],
                    [current_date, instance_name] + latency
                )
            elif description == 'CIT response out - API request out':
                self.append_data_to_csv(
                    self.csv_info['overall_latency']['filename'],
                    [current_date, instance_name] + latency
                )
            
            # Write request count
            self.append_data_to_csv(
                self.csv_info['request_count']['filename'],
                [current_date, instance_name, req_count]
            )

        except Exception as e:
            print('error:', e)
            print('')

    def append_data_to_csv(self, filename, data):
        """Append a row of data to specified CSV file"""
        filepath = os.path.join(self.output_dir, filename)
        with open(filepath, 'a', newline='') as f:
            writer = csv.writer(f)
            writer.writerow(data)

def main():
    # Initialize processor
    output_dir = '/path/to/output'
    processor = LatencyProcessor(output_dir)
    
    # Create initial files if they don't exist
    processor.create_initial_files()
    
    # Example usage:
    if '-c' in sys.argv:
        try:
            # Process CIT latency data
            processor.save_stats(
                CIT_LATENCY_ARR, 
                args.all_messages_log, 
                'CIT internal: Response out - Request in',
                args.instance_name
            )
            
            # Process total latency data
            processor.save_stats(
                TOTAL_LATENCY_ARR, 
                args.all_messages_log, 
                'CIT response out - API request out',
                args.instance_name
            )
            
            print("####### ALL DONE #######")
            
        except Exception:
            traceback.print_exc()

if __name__ == "__main__":
    main()
